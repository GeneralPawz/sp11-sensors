# sp11-sensors

Accelerometer (screen rotation), ambient light and compass for the Microsoft Surface Pro 11
(Snapdragon X) on Linux.

The sensors are run by the Qualcomm Sensor Core (SSC) on the ADSP. Linux talks to it with
[libssc](https://codeberg.org/DylanVanAssche/libssc) and
[iio-sensor-proxy](https://gitlab.freedesktop.org/hadess/iio-sensor-proxy) (`ssc-support`), but the
firmware first needs its configuration, served from Linux by `hexagonrpcd`
([hexagonrpc](https://github.com/linux-msm/hexagonrpc)).

This firmware maintains its own sensor registry: it deletes stale entries, writes each one to a temporary
file and renames it into place. That needs hexagonrpcd with a writable persist directory (`-P`), large
input buffers and the write methods of the file service: branch
[`ssc-writable-persist`](https://github.com/GeneralPawz/hexagonrpc/tree/ssc-writable-persist).

| File | Purpose |
|---|---|
| `sp11-sensors-config` | Installs Microsoft's sensor configuration (driver package `surfacepro_snscfgcrd8380`) to `/usr/share/qcom/x1p64100/Microsoft/denali`. Downloaded from the WOA-Project driver repository, or `--from` a Windows driver copy. The files are not redistributed here. |
| `sp11-sensors.service` | Runs `hexagonrpcd -s -P /var/lib/sp11-sensors/persist` for the ADSP sensor domain |
| `81-sp11-sensors.rules` | Enables the SSC accelerometer in iio-sensor-proxy (off by default upstream) and sets its mount matrix |
| `sp11-autorotate` + `sp11-autorotate.service` | User service for Hyprland: rotates the internal screen, touchscreen and pen with the orientation. `touch ~/.config/sp11-autorotate/disabled` keeps the screen as it is |

Details that matter on this device:

- `sns_reg_config` and `json.lst` ship with CRLF line endings; the firmware keeps the `\r` in paths
  (`.../registry\r`), so they are converted.
- `/sys/devices/soc0` as seen by the firmware carries Windows' values (`hw_platform=CRD`, `soc_id=615`),
  which select the `8380_crd_*` configurations.
- Factory calibration files (`factory_color_calibration.bin`, `acs_multiplier.bin`, `tdm_uid.bin`,
  `dev_runtime_mod.bin`) are requested but not part of the driver package; the sensors work without them.

Tested on a Surface Pro 11, X1P64100, LCD: accelerometer, magnetometer, compass and light sensor via `ssccli`.
