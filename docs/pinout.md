# Pinout y conexiones — ZMK Charybdis

Referencia de las conexiones eléctricas definidas por este módulo ZMK para la
BastardKB Charybdis. Las citas `archivo:línea` apuntan a la rama `main` del
repositorio y son **indicativas**: pueden desviarse a medida que los archivos
cambian.

> **Fuente de verdad**: los overlays y `.dtsi` del shield, no el esquema de la PCB.
> Ante cualquier duda, contrastar con el esquema oficial de BastardKB (ver
> [Pendientes de verificar](#pendientes-de-verificar)).

---

## 1. Matriz de teclas

Definida de forma idéntica en izquierda y derecha:

- `boards/shields/charybdis/charybdis_left.overlay:4-23`
- `boards/shields/charybdis/charybdis_right.overlay:13-31`

Ambos lados usan **los mismos pines**. Lo que cambia entre mitades es el
`col-offset` del transform y el rol central/periférico, nunca el cableado.

### Columnas

| Índice | Pin `pro_micro` | D | GPIO nRF52840 | Flags |
| --- | --- | --- | --- | --- |
| COL 0 | `pro_micro 19` | D19 | P0.02 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 1 | `pro_micro 20` | D20 | P0.29 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 2 | `pro_micro 10` | D10 | P0.09 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 3 | `pro_micro 6` | D6 | P1.00 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 4 | `pro_micro 7` | D7 | P0.11 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |
| COL 5 | `pro_micro 8` | D8 | P1.04 | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### Filas

| Índice | Pin `pro_micro` | D | GPIO nRF52840 | Flags |
| --- | --- | --- | --- | --- |
| ROW 0 | `pro_micro 18` | D18 | P1.15 | `GPIO_ACTIVE_HIGH` |
| ROW 1 | `pro_micro 5` | D5 | P0.24 | `GPIO_ACTIVE_HIGH` |
| ROW 2 | `pro_micro 4` | D4 | P0.22 | `GPIO_ACTIVE_HIGH` |
| ROW 3 | `pro_micro 9` | D9 | P1.06 | `GPIO_ACTIVE_HIGH` |

La columna **GPIO nRF52840** proviene del mapa `pro_micro` de la placa
**nice_nano_v2** en ZMK (`app/boards/arm/nice_nano/arduino_pro_micro_pins.dtsi`):
el mapeo depende de la placa, y `pro_micro N` es un número de pin D, no un número
de GPIO.

### Detalles de la matriz

| Propiedad | Valor | Fuente |
| --- | --- | --- |
| Dirección de diodos | `row2col` | `charybdis.dtsi:37` |
| Transform 6 columnas | 12 cols × 4 rows | `charybdis.dtsi:10-21` |
| Transform 5 columnas | 10 cols × 4 rows | `charybdis.dtsi:23-34` |
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
del nRF52840. A diferencia de la matriz, el sensor se direcciona **directamente**
con `&gpio0` / `NRF_PSEL` en lugar del conector `&pro_micro`; físicamente usa los
mismos pines que las posiciones `pro_micro` D0-D3.

| Señal SPI | nRF52840 | `pro_micro` equivalente | Definición |
| --- | --- | --- | --- |
| SCK | P0.08 | `pro_micro 0` (D0) | `charybdis_3610.dtsi:17` |
| MOSI | P0.17 | `pro_micro 2` (D2) | `charybdis_3610.dtsi:18` |
| MISO | P0.17 | `pro_micro 2` (D2) | `charybdis_3610.dtsi:19` |
| CS | P0.20 | `pro_micro 3` (D3) | `cs-gpios` — `charybdis_3610.dtsi:39` |
| IRQ (MOT) | P0.06 | `pro_micro 1` (D1) | `irq-gpios` — `charybdis_3610.dtsi:46` |

### Parámetros del sensor

| Propiedad | Valor | Fuente |
| --- | --- | --- |
| `spi-max-frequency` | `2000000` (2 MHz) | `charybdis_3610.dtsi:45` |
| `cpi` | `3200` | `charybdis_3610.dtsi:47` |
| `evt-type` | `INPUT_EV_REL` | `charybdis_3610.dtsi:48` |
| Salidas | `INPUT_REL_X`, `INPUT_REL_Y` | `charybdis_3610.dtsi:49-50` |
| IRQ flags | `GPIO_ACTIVE_LOW \| GPIO_PULL_UP` | `charybdis_3610.dtsi:46` |

### Pines compartidos entre sensor y matriz

Con el mapeo correcto de nice_nano_v2, el sensor y la matriz del **lado derecho**
(el único que activa el trackball) ocupan conjuntos de GPIO **disjuntos**:

- Sensor (SPI0): {P0.06, P0.08, P0.17, P0.20}
- Matriz (lado derecho): {P0.02, P0.29, P0.09, P1.00, P0.11, P1.04, P1.15, P0.24, P0.22, P1.06}

**No hay pines compartidos: no es un conflicto real ni un error de adaptación.**
La aparente colisión de versiones anteriores de este documento era un artefacto
de confundir el índice `pro_micro` con el número de GPIO: `pro_micro N` **no** es
`P0.N`. Con el mapeo correcto, ningún GPIO del sensor coincide con un GPIO de la
matriz.

**El overlay es idéntico al upstream.** Un `diff -ru` del directorio completo
`boards/shields/charybdis/` contra
[`victorlucachi/zmk-keyboards-charybdis`](https://github.com/victorlucachi/zmk-keyboards-charybdis)
da **cero diferencias**. `config/charybdis.keymap` y `config/charybdis.conf`
también son idénticos. No hubo "copia sin adaptar": este repo es el upstream
tal cual.

> El grupo `MATRIX`/`TRCK` del esquema de BastardKB pertenece a la Splinktegrated
> RP2040, un MCU distinto del nice!nano; no aporta evidencia sobre estos GPIO. La
> conclusión "no es un conflicto real" se sostiene por el mapeo correcto: los
> conjuntos de pines son disjuntos.

### MOSI y MISO en el mismo pin (P0.17)

Es **deliberado y correcto**. El pin `SDIO` del PMW3610 es bidireccional y el
[README del driver de badjeff](https://github.com/badjeff/zmk-pmw3610-driver)
publica exactamente esta configuración como ejemplo canónico:

```dts
psels = <NRF_PSEL(SPIM_SCK,  0, 8)>,
        <NRF_PSEL(SPIM_MOSI, 0, 17)>,
        <NRF_PSEL(SPIM_MISO, 0, 17)>;
```

Este repo lo replica textualmente (`charybdis_3610.dtsi:17-19`), incluido
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
| `charybdis_dongle` | **central** (`Kconfig.defconfig:6-7`) | `mock_kscan` (`charybdis_dongle.overlay`) | — | `CONFIG_INPUT=y`, pointing + smooth scrolling, display OLED SH1106 (`dongle_display`) |
| `charybdis_left` | periférico | matriz GPIO | deshabilitado | `CONFIG_INPUT=y` |
| `charybdis_right` | periférico | matriz GPIO | **habilitado** | SPI + PMW3610 + pointing + ext_power |

`ZMK_SPLIT` se habilita por defecto para los tres shields
(`Kconfig.defconfig:11-16`).

**Detalle de activación del trackball** (`charybdis_right.overlay:33-39`):

```dts
&spi0    { status = "okay"; };
&trackball { status = "okay"; };
&trackball_split { device = <&trackball>; };
```

**En el lado izquierdo** (`charybdis_left.overlay:25-27`) el sensor se desactiva
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

`charybdis.dtsi:162-170` define el nodo de transporte:

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
(`charybdis_right.overlay:41-43`); el dongle central lo recibe como *input split*
y lo reemite como puntero.

---

## 5. Cableado físico (resumen de armado)

| Tramo | Conductores |
| --- | --- |
| Interconexión entre mitades | TRRS de 4 conductores (VCC, GND, datos, reloj del split serial) + detección `EXT_POWER` si hay batería. El repo no define pines TRRS extra: usa el default del shield. |
| Matriz de teclas | Propia de cada mitad (6×4 independiente). **No hay cables de matriz que crucen** entre mitades. |
| Trackball | 5 hilos al lado derecho: SCK, SDIO, CS, MOT/IRQ, +3V3 y GND. |
| Dongle | Sin matriz ni sensor: solo radio, USB y la pantalla OLED I2C (sección 6). |

### Cableado del dongle

El dongle es un `nice_nano_v2` (`build.yaml`) que actúa como **central** del split
(`Kconfig.defconfig:6-7`) y alimenta a **2 periféricos BLE**
(`CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2` en `charybdis_dongle.conf`). Se
configura con TX a +8 dBm (`CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`) y
`CONFIG_ZMK_SLEEP=n`: va siempre alimentado por USB.

| Tramo | Conductores |
| --- | --- |
| Alimentación / datos | USB-C: alimenta el controlador y transporta HID y ZMK Studio. |
| Matriz de teclas | Ninguna: usa un kscan simulado (`zmk,kscan = &mock_kscan` en `charybdis_dongle.overlay`). |
| Trackball | Ninguno: el sensor PMW3610 solo existe en la mitad derecha. |
| TRRS | Ninguno. |
| OLED 1.3" SH1106 | 4 hilos por I2C: `VCC` → 3V3, `GND` → GND, `SDA` → P0.17 (D2), `SCL` → P0.20 (D3). |

La pantalla cuelga del bus `pro_micro_i2c` (= `&i2c0`) con dirección `0x3c`
(`charybdis_dongle.overlay:17-34`); se habilita con `CONFIG_ZMK_DISPLAY=y` y se
apaga por inactividad con `CONFIG_ZMK_IDLE_TIMEOUT=60000`
(`charybdis_dongle.conf:12-13`). El objetivo de build combina los shields
`charybdis_dongle dongle_display` con el snippet `studio-rpc-usb-uart`
(`build.yaml:21-24`).

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
| Pines compartidos sensor/matriz | **No es problema** | No hay pines compartidos: con el mapeo correcto de nice_nano_v2 los conjuntos de GPIO del sensor y de la matriz son disjuntos; el overlay es idéntico al upstream (`diff -ru` vacío) |
| MOSI/MISO en P0.17 | **Correcto y deliberado** | Ejemplo canónico del README del driver badjeff; `SDIO` es bidireccional |
| Pinout de la matriz | **Idéntico al upstream** | `diff -ru boards/shields/charybdis/` sin diferencias |
| Pinout del trackball | **Idéntico al upstream** | `charybdis_3610.dtsi` sin diferencias |

### Pendiente real detectado

**`config/west.yml` no declara el módulo del shield.** El árbol local define los
shields en `boards/shields/charybdis/`, pero `west.yml` solo incluye `zmk`,
`zmk-pmw3610-driver` y `englmaxi/zmk-dongle-display` (`revision: v0.3`, el
módulo externo del display) — le falta el proyecto del módulo del shield que el
propio README indica como necesario en configuraciones tipo *template*:

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
3. **Controlador del dongle**: `build.yaml` usa únicamente `nice_nano_v2` para el
   dongle. El README advierte explícitamente que flashear el reset equivocado
   puede bloquear el controlador; verificar el nombre del `.uf2` antes de
   copiarlo.
