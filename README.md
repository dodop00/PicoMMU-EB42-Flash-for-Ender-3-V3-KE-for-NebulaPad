# Firmware + config for using **EBB42 / Pico MMU** with **Ender-3 V3 KE + Nebula Pad** (Creality “dirty” Klipper fork)

This repo provides a **known-working firmware binary** for the EBB42/Pico MMU and the **extra Klipper Python modules** needed on Creality’s Nebula Pad. It’s aimed at avoiding version-mismatch errors (e.g., `can't convert negative value to unsigned`) and getting CAN/USB comms stable.

---

## 🧠 Version compatibility

Built to match Creality’s dirty Klipper fork reported as:

    09faed31-dirty

Use the included binary as-is for the EBB42/Pico MMU MCU.

---

## 📂 Folder contents

- `firmware/` — prebuilt MCU firmware binary for EBB42/Pico MMU  
- `extras/` — Python modules to copy into the Nebula Pad’s `klippy/extras/` (e.g., `prtouch_v2.py`, `z_compensate.py`, `mcu_temperature.py`, etc.)  
- `config-example/` — my messy config to serve as an example (no judging my mess) after yours looks like this put it in the config backup in /usr/data/... or /usr/share/klipper_config/


---

## 🔧 Flashing the Pico MMU (EBB42) via DFU (from your PC)

Most EBB42 boards use **STM32G0B1** with a **32 KiB bootloader**. The firmware file below assumes that.

DFU Util v0.9: https://sourceforge.net/projects/dfu-util/files/

**Enter DFU mode**
1. Unplug USB from the EBB42  
2. Hold **BOOT**, tap **RESET**, release **BOOT**  
3. Device should appear as **STM32 DFU** (`0483:df11`)

**Verify DFU is detected**
    
    dfu-util -l

You should see an entry with `@Internal Flash`.

**Flash the firmware**  
Most common (32 KiB bootloader):

    dfu-util -a 0 -s 0x08008000:leave -D firmware/klipper-ebb42.bin

If it keeps returning to DFU (no bootloader case):

    dfu-util -a 0 -s 0x08000000:leave -D firmware/klipper-ebb42.bin

Power-cycle the board after flashing.

If it still boots to DFU, confirm your boot jumper/pad (**BOOT0**) isn’t stuck high, and recheck the offset you used.

---

## 📦 Installing the extra Klipper modules on the Nebula Pad (FileZilla / SFTP)

These enable Creality-specific features (PRTouch v2, z-compensate, MCU temp, etc.).

1. Open **FileZilla** → **File > Site Manager…**  
   - **Protocol:** SFTP  
   - **Host:** *Nebula Pad IP*  
   - **Port:** 22  
   - **Logon Type:** Normal (use your pad’s SSH user/pass)
2. On the **remote** side, navigate to:

       /usr/share/klipper/klippy/extras/

3. On the **local** side, open this repo’s `extras/` folder.
4. **Drag & drop** the desired `.py` files into the remote `extras/` directory.  
   Tip: back up originals first, e.g. rename `file.py` → `file.py.bak`.
5. In Fluidd/Mainsail **Console**, run:

       RESTART

   (or reboot the pad) to load the new modules.

---

## 🧩 Add to your config

    # MCU definition (rename 'pico_mmu' to your preference)
    [mcu pico_mmu]
    # If using CAN (recommended):
    # canbus_uuid: <YOUR_CAN_UUID>
    
    # If using USB instead of CAN (after leaving DFU):
    # serial: /dev/ttyACM0   # adjust as needed
    
    [resonance_tester]
    accel_chip: adxl345
    accel_per_hz: 70
    # Use standard probe_points (not prtouch_probe_points) unless your host fork supports it
    probe_points:
      117.5, 117.5, 100
    max_freq: 90

---

## Notes

- Find the CAN UUID with `canbus_query.py` on the pad.  
- If `/dev/serial/by-id/` doesn’t exist on the pad, identify the port with:

      ls /dev/ttyACM*
      dmesg | tail -n 50
