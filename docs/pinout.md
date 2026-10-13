# Pinout y conexiones — ZMK Charybdis

Referencia de las conexiones eléctricas definidas por este módulo ZMK para la
BastardKB Charybdis. Todas las citas `archivo:línea` apuntan al estado actual del
repositorio (`master`, commit `cce97e6`).

> **Fuente de verdad**: los overlays y `.dtsi` del shield, no el esquema de la PCB.
> Ante cualquier duda, contrastar con el esquema oficial de BastardKB (ver
> [Pendientes de verificar](#pendientes-de-verificar)).

---

## 1. Matriz de teclas

Definida de forma idéntica en izquierda y derecha:

- `boards/shields/charybdis/charybdis_left.overlay:6-25`
- `boards/shields/charybdis/charybdis_right.overlay:9-28`

Ambos lados usan **los mismos pines**. Lo que cambia entre mitades es el
`col-offset` del transform y el rol central/periférico, nunca el cableado.

### Columnas

| Índice | Pin `pro_micro` | GPIO nRF52840 | Flags |
| --- | --- | --- | --- |
| COL 0 | `pro_micro 19` | P0.19 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 1 | `pro_micro 20` | P0.20 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 2 | `pro_micro 10` | P0.10 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 3 | `pro_micro 6` | P0.06 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 4 | `pro_micro 7` | P0.07 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 5 | `pro_micro 8` | P0.08 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### Filas

| Índice | Pin `pro_micro` | GPIO nRF52840 | Flags |
| --- | --- | --- | --- |
| ROW 0 | `pro_micro 18` | P0.18 | `GPIO_ACTIVE_HIGH` |
| ROW 1 | `pro_micro 5` | P0.05 | `GPIO_ACTIVE_HIGH` |
| ROW 2 | `pro_micro 4` | P0.04 | `GPIO_ACTIVE_HIGH` |
| ROW 3 | `pro_micro 9` | P0.09 | `GPIO_ACTIVE_HIGH` |

### Detalles de la matriz

| Propiedad | Valor | Fuente |
| --- | --- | --- |
| Dirección de diodos | `row2col` | `charybdis.dtsi:35` |
| Transform 6 columnas | 12 cols × 4 rows | `charybdis.dtsi:12-24` |
| Transform 5 columnas | 10 cols × 4 rows | `charybdis.dtsi:27-39` |
| `col-offset` derecha (6 col) | `6` | `charybdis_right.overlay:5-7` |
| `col-offset` derecha (5 col) | `5` | `charybdis_right.overlay:9-11` |

**Offsets por lado**

- **Izquierda**: RC(0-3, 0-5), sin offset.
- **Derecha**: `col-offset = <6>` (6 columnas) o `<5>` (5 columnas). El `map` del
  transform ya viene espejado, así que la matriz física local es la misma en
  ambos lados y el offset solo desplaza el índice lógico.

**Variante de 5 columnas**: usa exactamente los mismos pines. La columna 0 se
ignora por definición del `five_column_transform`; no hay que recablear nada.

---

## 2. Trackball PMW3610 — solo lado derecho

Definido en `boards/shields/charybdis/charybdis_3610.dtsi`, sobre el puerto SPI0
del nRF52840. **Es el único bloque que usa pines fuera del estándar `pro_micro`.**

| Señal SPI | nRF52840 | `pro_micro` equivalente | Definición |
| --- | --- | --- | --- |
| SCK | P0.08 | `pro_micro 8` | `charybdis_3610.dtsi:20` |
| MOSI | P0.17 | `pro_micro 17` | `charybdis_3610.dtsi:21` |
| MISO | P0.17 | `pro_micro 17` | `charybdis_3610.dtsi:22` |
| CS | P0.20 | `pro_micro 20` | `cs-gpios` — `charybdis_3610.dtsi:33` |
| IRQ (MOT) | P0.06 | `pro_micro 6` | `irq-gpios` — `charybdis_3610.dtsi:40` |

### Parámetros del sensor

| Propiedad | Valor | Fuente |
| --- | --- | --- |
| `spi-max-frequency` | `2000000` (2 MHz) | `charybdis_3610.dtsi:38` |
| `cpi` | `3200` | `charybdis_3610.dtsi:41` |
| `evt-type` | `INPUT_EV_REL` | `charybdis_3610.dtsi:42` |
| Salidas | `INPUT_REL_X`, `INPUT_REL_Y` | `charybdis_3610.dtsi:43-44` |
| IRQ flags | `GPIO_ACTIVE_LOW \| GPIO_PULL_UP` | `charybdis_3610.dtsi:40` |

### Pines compartidos entre sensor y matriz

Los pines del sensor y varios pines de la matriz coinciden en el **lado derecho**,
el único que activa el trackball:

| Pin | Uso en el trackball | Uso en la matriz |
| --- | --- | --- |
| P0.20 | CS del sensor | COL 1 |
| P0.08 | SCK | COL 5 |
| P0.06 | IRQ / MOT | COL 3 |

**No es un conflicto real ni un error de adaptación.** Verificado contra fuentes
primarias:

1. **El overlay es idéntico al upstream.** Un `diff -ru` del directorio completo
   `boards/shields/charybdis/` contra
   [`victorlucachi/zmk-keyboards-charybdis`](https://github.com/victorlucachi/zmk-keyboards-charybdis)
   da **cero diferencias**. `config/charybdis.keymap` y `config/charybdis.conf`
   también son idénticos. No hubo "copia sin adaptar": este repo es el upstream
   tal cual.
2. **`pro_micro N` y `P0.NN` son el mismo pin físico.** La aparente colisión
   surge de comparar dos notaciones de la misma cosa: `pro_micro 8` *es* P0.08,
   según el mapeo de la [wiki nRFMicro](https://github.com/joric/nrfmicro/wiki/Pinout)
   y el [pinout nice!nano v2](https://www.xopx.de/nrf52840-pro-micro-nicenano-v2-0-pinout/)
   (D0=P0.08, D1=P0.06, D2=P0.17, D3=P0.20). El lado derecho usa exactamente los
   mismos pines de matriz que el izquierdo; el hardware simplemente comparte esos
   pines entre el barrido de teclas y el bus SPI del sensor.
3. **La PCB separa los grupos.** El esquema oficial de BastardKB documenta
   `MATRIX` y `TRCK` como grupos de pines **distintos** en la Splinktegrated
   ([docs.bastardkb.com/hw/pins.html](https://docs.bastardkb.com/hw/pins.html)).
   El mapeo RB2040 de referencia es: `SS`=GP16, `MO`=GP23, `SC`=GP22,
   `MI`=GP20, y las columnas `C1`-`C6` en otros GPIO.

### MOSI y MISO en el mismo pin (P0.17)

Es **deliberado y correcto**. El pin `SDIO` del PMW3610 es bidireccional y el
[README del driver de badjeff](https://github.com/badjeff/zmk-pmw3610-driver)
publica exactamente esta configuración como ejemplo canónico:

```dts
psels = <NRF_PSEL(SPIM_SCK,  0, 8)>,
        <NRF_PSEL(SPIM_MOSI, 0, 17)>,
        <NRF_PSEL(SPIM_MISO, 0, 17)>;
```

Este repo lo replica textualmente (`charybdis_3610.dtsi:20-22`), incluido
`cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>` e `irq-gpios = <&gpio0 6 ...>`. Una
configuración independiente verificada publica el mismo mapeo lógico
(SCK=P0.08, MOSI/MISO=P0.17, CS=P0.20, IRQ=P0.06) en
[syafiqdanial/zmk-config-charybdis-nano-legacy](https://github.com/syafiqdanial/zmk-config-charybdis-nano-legacy).

> **Estado**: descartado como problema. La hipótesis anterior de "pines mal
> copiados" quedó refutada por el diff vacío contra el upstream y por el ejemplo
> del driver. Lo que **no** verificó ninguna fuente es la orientación física del
> zócalo de la PCB: eso solo se confirma en hardware.

---

## 3. Reparto de roles por build

Configurado en `boards/shields/charybdis/Kconfig.defconfig` y los `.conf` de cada
shield.

| Build | `ZMK_SPLIT_ROLE` | Kscan | Trackball | Input / Pointing |
| --- | --- | --- | --- | --- |
| `charybdis_dongle` | **central** (`Kconfig.defconfig:4-7`) | `mock_kscan` (`charybdis_dongle.overlay`) | — | `CONFIG_INPUT=y`, pointing + smooth scrolling, display OLED SH1106 (`dongle_display`) |
| `charybdis_left` | periférico | matriz GPIO | deshabilitado | `CONFIG_INPUT=y` |
| `charybdis_right` | periférico | matriz GPIO | **habilitado** | SPI + PMW3610 + pointing + ext_power |

`ZMK_SPLIT` se habilita por defecto para los tres shields
(`Kconfig.defconfig:11-15`).

**Detalle de activación del trackball** (`charybdis_right.overlay:30-40`):

```dts
&spi0    { status = "okay"; };
&trackball { status = "okay"; };
&trackball_split { device = <&trackball>; };
```

**En el lado izquierdo** (`charybdis_left.overlay:27-29`) el sensor se desactiva
explícitamente:

```dts
&trackball_split { status = "disabled"; };
```

> El izquierdo **incluye** `charybdis_3610.dtsi` (`charybdis_left.overlay:2`) pero
> desactiva el nodo. La definición del sensor existe en ambos lados; solo el
> derecho la habilita. Los pines del sensor quedan reservados también en la
> izquierda aunque no se usen.

**Parámetros de consumo** (`charybdis_right.conf`): tiempos de muestreo del PMW3610
en reposo 100/200/300 ms (REST1/2/3), downshift de 500 ms en RUN y 3000/30000 ms
en REST1/REST2.

---

## 4. Transporte del trackball a través del split

`charybdis.dtsi:160-168` define el nodo de transporte:

```dts
split_inputs {
    trackball_split: trackball_split@0 {
        compatible = "zmk,input-split";
        reg = <0>;
    };
};
```

Ruta completa del movimiento del puntero:

```
sensor PMW3610  →  periférico derecho  →  BLE  →  dongle central  →  USB/BLE  →  host
```

El lado derecho engancha el sensor con `device = <&trackball>`
(`charybdis_right.overlay:39-40`); el dongle central lo recibe como *input split*
y lo reemite como puntero.

---

## 5. Cableado físico (resumen de armado)

| Tramo | Conductores |
| --- | --- |
| Interconexión entre mitades | TRRS de 4 conductores (VCC, GND, datos, reloj del split serial) + detección `EXT_POWER` si hay batería. El repo no define pines TRRS extra: usa el default del shield. |
| Matriz de teclas | Propia de cada mitad (6×4 independiente). **No hay cables de matriz que crucen** entre mitades. |
| Trackball | 5 hilos al lado derecho: SCK, SDIO, CS, MOT/IRQ, +3V3 y GND. |
| Dongle | Sin matriz ni sensor: solo radio + USB. |

---

## 6. OLED del dongle — 1.3" I2C (SH1106)

El dongle incorpora una pantalla OLED de 1.3" con controlador SH1106 sobre el bus
I2C. La lógica de presentación (capa activa, modificadores, batería del
periférico, salida y bongo cat) proviene del módulo externo
[englmaxi/zmk-dongle-display](https://github.com/englmaxi/zmk-dongle-display),
declarado en `config/west.yml` con `revision: v0.3`. El módulo aporta el shield
`dongle_display`; el nodo de hardware del panel se define en nuestro overlay
`boards/shields/charybdis/charybdis_dongle.overlay`.

El panel se conecta al bus `pro_micro_i2c` (= `&i2c0` del nRF52840), el mismo
expuesto por el estándar `pro_micro` del nice!nano v2. En el dongle ese bus queda
libre porque no se incluye `charybdis_3610.dtsi` (el trackball solo vive en el
lado derecho).

| Señal I2C | GPIO nRF52840 | `pro_micro` / nice!nano |
| --- | --- | --- |
| SDA (datos) | P0.17 | `&pro_micro 2` / D2 |
| SCL (reloj) | P0.20 | `&pro_micro 3` / D3 |

Parámetros del controlador en `charybdis_dongle.overlay`:

| Propiedad | Valor |
| --- | --- |
| Dirección I2C (`reg`) | `0x3c` |
| `compatible` | `sinowealth,sh1106` |
| Resolución | 128 × 64 |
| `segment-offset` | `<2>` (requerido por el panel de 1.3") |
| `multiplex-ratio` | `<63>` |
| Precharge | `prechargep = <0x22>` |

> **Nota sobre los colores**: el nodo incluye la propiedad `inversion-on`. Si el
> panel muestra los colores invertidos en el hardware real, esa propiedad puede
> eliminarse del overlay. El resto de las propiedades coincide con el binding de
> Zephyr 3.5.0 (`dts/bindings/display/sinowealth,sh1106-i2c.yaml`).

El dongle habilita el subsistema de display en `charybdis_dongle.conf`
(`CONFIG_ZMK_DISPLAY=y`) y define un tiempo de apagado por inactividad
(`CONFIG_ZMK_IDLE_TIMEOUT=60000`). El objetivo de build correspondiente combina
los shields `charybdis_dongle dongle_display` (ver `build.yaml`).

---

## Estado de verificación

Investigación cerrada contra fuentes primarias con estos resultados:

| Ítem | Veredicto | Evidencia |
| --- | --- | --- |
| Pines compartidos sensor/matriz | **No es problema** | El overlay es idéntico al upstream (`diff -ru` vacío); `pro_micro N` y `P0.NN` son el mismo pin; la PCB separa `MATRIX` y `TRCK` |
| MOSI/MISO en P0.17 | **Correcto y deliberado** | Ejemplo canónico del README del driver badjeff; `SDIO` es bidireccional |
| Pinout de la matriz | **Idéntico al upstream** | `diff -ru boards/shields/charybdis/` sin diferencias |
| Pinout del trackball | **Idéntico al upstream** | `charybdis_3610.dtsi` sin diferencias |

### Pendiente real detectado

**`config/west.yml` no declara el módulo del shield.** El árbol local define los
shields en `boards/shields/charybdis/`, pero `west.yml` solo incluye `zmk` y
`zmk-pmw3610-driver` — le falta el proyecto del módulo que el propio README
indica como necesario en configuraciones tipo *template*:

```yaml
- name: zmk-keyboards-charybdis
  url: https://github.com/victorlucachi/zmk-keyboards-charybdis
  revision: main
  path: modules/zmk-keyboards-charybdis
```

Es coherente con este repo alojar el módulo *in-tree* (los shields ya están
acá y `zephyr/module.yml` existe), pero conviene confirmar que la intención es
ser módulo autocontenido y no config-dependiente. **No tocar sin confirmar**
cuál de los dos modos es el buscado: cambiarlo altera el build.

### Pendientes de hardware

1. **Orientación física del zócalo del sensor**: ninguna fuente puede validarla;
   solo se confirma en la PCB real.
2. **Validación funcional**: si el trackball falla, revisar primero el delay de
   arranque del driver (`CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS`), que el
   propio README documenta como causa típica de `Incorrect product id 0xFF`.
3. **Controlador del dongle**: `build.yaml` incluye `seeeduino_xiao_ble` y
   `nice_nano_v2` para el dongle. El README advierte explícitamente que flashear
   el reset equivocado puede bloquear el controlador; verificar el nombre del
   `.uf2` antes de copiarlo.
