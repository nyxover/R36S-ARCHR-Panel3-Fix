# R36S ARCHR — Panel 3 Screen Fix

## Problem

R36S V12 (2023-08-18) clone with ARCHR OS (kernel 6.12.114) — screen shows
vertical bands (black, blue, white) instead of a clean image. Sound works.

The root cause: ARCHR's stock DTB (`rk3326-gameconsole-r36s.dtb`) uses the
wrong panel (`gameconsole,r36s-panel` / `sitronix,st7703` with built-in init).
This panel clone needs the **Panel 3** timing (porches 150/4/150, 944×512,
29 MHz, 60 Hz) and the **773-byte init sequence** from the original AURKNIX OS.

## Solution

A merged DTB that injects the panel description (official ARCHR
`archr,generic-dsi` format) into ARCHR's base DTB:

- **`rk3326-gameconsole-r36s-panel3.dtb`** (71953 bytes)
  - `compatible = "archr,generic-dsi"` + `panel_description`
  - Init sequence: 188 commands (938 bytes, from the `adjust` DTB)
  - Porches: 150/4/150, hsync-len 4
  - Flags: 0xE03
  - Delays: 2, 1, 20, 120, 20
  - Native mode: 60 Hz exact

## How to install

1. Copy `rk3326-gameconsole-r36s-panel3.dtb` to `I:\dtbs\`
2. Edit `I:\extlinux\extlinux.conf`:
   ```
   FDT /dtbs/rk3326-gameconsole-r36s-panel3.dtb
   # FDTOVERLAYS /overlays/mipi-panel.dtbo
   ```
3. Eject, reinsert, boot.

## Files

### dtbs/ (current working)
- `rk3326-gameconsole-r36s-panel3.dtb` — **merged DTB (active fix)**
- `rk3326-gameconsole-r36s.dtb` — ARCHR stock (wrong panel)
- `rk3326-r36s-panel3-adjust.dtb` — old OS DTB (938-byte init, native-mode→/adjust)
- `rk3326-r36s-panel3-porches.dtb` — old OS DTB (773-byte init, 150/4/150)
- `rk3326-r36s-panel3-flags.dtb` — old OS DTB (773-byte init, flags 0xE03)
- `uboot-panel3.dtb` — u-boot DTB (same panel correction)
- `rg351mp-uboot.dtb`, `rg351p-uboot.dtb`, `rg351v-uboot.dtb` — u-boot variants

### dtbs/backup/ (old OS saved DTBs)
- All 13 ArchR DTBs + 3 panel3 variants + 4 u-boot DTBs

### dtbs/prebuilt-rocknix.dtb
- Pre-built merged DTB (rocknix,generic-dsi, 201 lines, 188 init commands)

### dtbs/merged-archr.dtb
- My merged DTB (archr,generic-dsi, 201 lines, 188 init commands) — the one that works

### scripts/
- `install-panel3-simple.ps1` — installation script
- `verify-phandle.ps1` — DTB verification script

## Background

- Console: R36S V12 2023-08-18, 944×512 @ 29 MHz, porches 150/4/150
- Old OS: AURKNIX-RK3326 (kernel 4.4.189) — used `rk3326-r36s-panel3-adjust.dtb`
- New OS: ARCHR (kernel 6.12.114) — uses `panel_description` format
- The fix: convert the old panel timing/init into ARCHR's `panel_description` format
  and merge it into ARCHR's base DTB (no u-boot overlay needed)

## Author

nyxover + community

## License

MIT
