# Quickstart (single-sensor fork): required steps only

## Prerequisites
- Klipper + Moonraker running (`~/klipper`, `~/printer_data/config`). Python 3 comes with Klipper's venv; no extra pip packages.
- Mainboard with: 1 toolhead endstop (hotend attached), servo pin, shared extruder stepper, N heaters/thermistors, Klicky switch pin.
- Do not install or restart during a print.

## 1. Install the Klipper modules and config
```bash
git clone https://github.com/JoseMPena/MedusaHC.git ~/MedusaHC-source
install -m 0644 ~/MedusaHC-source/klippy/extras/medusahc.py  ~/klipper/klippy/extras/
install -m 0644 ~/MedusaHC-source/klippy/extras/pin_watch.py ~/klipper/klippy/extras/
mkdir -p ~/printer_data/config/MedusaHC
cp -a ~/MedusaHC-source/config/MedusaHC/. ~/printer_data/config/MedusaHC/
cp ~/MedusaHC-source/examples/printer-config/klicky_config.cfg ~/printer_data/config/
```
Do not use `install-online.sh`: it pulls the upstream repo, not this fork.

## 2. Edit `MedusaHC/MHC_config.cfg`
- `pin_e` = toolhead endstop pin. Keep **no** `pin_t*` lines. Set `tool_count`.
- Set pins/thermistors/PID for `[extruder]`..`[extruderN]`, `[servo my_servo]`, and `[duplicate_pin_override]` (shared extruder step/dir/enable + `pin_e`).
- Remove duplicate `[extruder*]` sections from `printer.cfg`.

## 3. Edit `MedusaHC/MHC_variables.cfg`
- `max_tool`, `tools_direction` (1 V-Front / -1 V-Back), dock X/Y coordinates, latch/prime/brush positions.
- Keep `[save_variables]` (stores offsets and `mhc_mounted_tool`).

## 4. `printer.cfg`
```ini
[include MedusaHC/MHC_config.cfg]
[include klicky_config.cfg]
```
- Provide a `_HOME_REQUEST` macro (copy from `config/MedusaHC/macros_examples.cfg`).
- Provide `PAUSE`/`RESUME` per `docs/recovery.md`.
- Probe: `endstop_pin: probe:z_virtual_endstop` on `[stepper_z]`.
- Edit `klicky_config.cfg`: probe `pin`, `_KLICKY_CFG` `cal_x/cal_y` (nozzle over switch). Set `use_dock: 1` only if Klicky attach/dock macros exist.

## 5. First start
1. Firmware restart while idle.
2. Fit a hotend: `MHC_ASSIGN_TOOL T=<n>` (only needed if state unknown). Empty head: `current_tool` = -1.
3. `PROBE_CALIBRATE` for Klicky `z_offset`.
4. `CALIBRATE_TOOL_Z` (auto Z offsets vs T0). XY offsets stay manual (`SAVE_VARIABLE VARIABLE=t1_gcode_x_offset VALUE=...`).
5. `TEST_TOOLS` at low speed with hand on power.

## Behavior to know
- Sensor says only "hotend attached". The mounted tool index is software state; a manual swap while attached needs `MHC_ASSIGN_TOOL T=<n>`.
- `MHC_SET`/`MHC_DROP` fail (pause print) if `pin_e` does not confirm attach/detach within `sensor_timeout`.
