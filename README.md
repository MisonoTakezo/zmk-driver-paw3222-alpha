# ZMK PAW3222 Driver

This driver enables use of the PIXART PAW3222 optical sensor with the ZMK framework.

---

## Features

- SPI communication with the PAW3222 sensor
- Supports cursor movement, vertical/horizontal scrolling, and snipe (precision) mode
- Layer-based input mode switching (move, scroll, snipe)
- Runtime CPI (resolution) adjustment
- Power management and low-power modes
- Optional power GPIO support

---

## Device Tree Configuration

Configure the sensor in your shield or board config file (`.overlay` or `.dtsi`):

```dts
&spi1 {
    status = "okay";
    cs-gpios = <&gpio0 17 GPIO_ACTIVE_LOW>;
    
    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,paw3222";
        reg = <0>;
        spi-max-frequency = <4000000>;
        irq-gpios = <&gpio0 15 GPIO_ACTIVE_LOW>;
        
        /* PAW3222 設定 */
        res-cpi = <800>;               /* メインCPI: 800 */
        snipe-cpi = <400>;             /* スナイプ時のCPI: 400 */
        snipe-layer = <1>;             /* スナイプレイヤー: 1 */
        scroll-layer = <3>;            /* スクロールレイヤー: 3 (Shift押下でズーム) */

        scroll-tick = <10>;            /* スクロール感度 */
        
        rotation = <0>;                /* 回転角度 (0, 90, 180, 270) */
        
        force-awake;                   /* 常時起動 */
    };
};
```

---

## Properties

| Property Name            | Type          | Required | Description                                                                                                               |
| ------------------------ | ------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| irq-gpios                | phandle-array | Yes      | GPIO connected to the motion pin, active low.                                                                             |
| power-gpios              | phandle-array | No       | GPIO connected to the power control pin.                                                                                  |
| res-cpi                  | int           | No       | CPI resolution for the sensor. Can also be changed at runtime using the `paw32xx_set_resolution()` API.                   |
| snipe-cpi                | int           | No       | CPI resolution for snipe mode. If not specified, uses the same value as res-cpi.                                         |
| force-awake              | boolean       | No       | Initialize the sensor in "force awake" mode. Can also be enabled/disabled at runtime via the `paw32xx_force_awake()` API. |
| rotation                 | int           | No       | Physical rotation of the sensor in degrees. (0, 90, 180, 270)                                                             |
| scroll-tick              | int           | No       | Threshold for scroll movement (delta value above which scroll is triggered).                                              |
| snipe-layer              | int           | No       | Layer number for snipe mode (high precision cursor movement).                                                             |
| scroll-layer             | int           | No       | Layer number for auto scroll mode. Hold Shift key during scroll mode for zoom functionality.                             |

---

## Usage

- The driver automatically switches input mode (move, scroll, snipe) based on the active ZMK layer and your devicetree configuration.
- In scroll mode (`scroll-layer`):
  - **Normal operation**: Automatically detects movement direction and scrolls accordingly
    - Larger vertical movement → vertical scroll
    - Larger horizontal movement → horizontal scroll
  - **Zoom operation**: Hold Shift key + vertical trackball movement for zoom
    - Up movement → zoom in
    - Down movement → zoom out
    - **On Mac**: Combine with Cmd key (Cmd+Shift+Scroll) for zoom functionality
- In snipe mode (`snipe-layer`), the sensor uses the specified `snipe-cpi` for high precision movement.
- You can adjust CPI (resolution) at runtime using the API (see below).
- Use `rotation` to match the sensor's physical orientation.
- Configure `scroll-tick` to tune scroll sensitivity.

---

## API Reference

### Change CPI (Resolution)

```c
int paw32xx_set_resolution(const struct device *dev, uint16_t res_cpi);
```

- Changes sensor resolution at runtime.

### Force Awake Mode

```c
int paw32xx_force_awake(const struct device *dev, bool enable);
```

- Enables/disables "force awake" mode at runtime.

---

## Troubleshooting

- If the sensor does not work, check SPI and GPIO wiring.
- Confirm `irq-gpios` and (if used) `power-gpios` are correct.
- Use Zephyr logging to check for errors at boot.
- Ensure the ZMK version matches the required version.

---

## License

```
SPDX-License-Identifier: Apache-2.0

Copyright 2024 Google LLC
Modifications Copyright 2025 sekigon-gonnoc
Modifications Copyright 2025 nuovotaka
```
