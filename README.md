# HP LaserJet 1020 — Modern macOS Driver (Under 4MB)

HP never released an official driver for the LaserJet 1020 Printer for modern macOS versions (Ventura, Sonoma, Sequoia, or macOS 26).

This repo is an ultra minimal, stripped-down, patched and community-maintained driver package that gets the HP LaserJet 1020 Printer fully working on modern macOS in under 4MB.

It works by extracting only the 4 files that actually matter from Apple's HP Printer Drivers 5.1.1 package, placing them in the right locations, and using the most-compatible LaserJet 1022 PPD (which drives the 1020 Printer hardware perfectly).

Tested and working on: **macOS Ventura, Sonoma, Sequoia, and macOS 26**.

---

## Why does this exist?

The HP LaserJet printers uses a proprietary print language called ZJS. Apple's HP driver package contains a CUPS filter (`rasterToHPZJS`) that handles the conversion — but the installer is hard-blocked to only run on macOS 12.0 or older. The printer hardware itself works fine, the drivers work fine, Apple just never updated the version check.

This repo skips all of that and gives you just the files you need to get it on 1020 family printers working.

---

## Files

| File                  | Size  | Purpose                                             |
| --------------------- | ----- | --------------------------------------------------- |
| `HP LaserJet 1022.gz` | 12KB  | PPD — printer config and capabilities               |
| `rasterToHPZJS`       | 1.3MB | CUPS filter — converts raster to ZJS print language |
| `commandToHPZJS`      | 60KB  | CUPS command filter                                 |
| `hp1020.acl`          | 76KB  | Firmware init file sent to the printer on each job  |

---

## Install

### 1. Clone this repo

```bash
git clone https://github.com/anxkhn/hp1020-driver-mac
cd hp1020-driver-mac
```

### 2. Place the files

```bash
# PPD
sudo cp "HP LaserJet 1022.gz" "/Library/Printers/PPDs/Contents/Resources/"

# Create the bundle structure
sudo mkdir -p "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/MacOS"
sudo mkdir -p "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/Resources"

# Filters
sudo cp rasterToHPZJS "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/MacOS/"
sudo cp commandToHPZJS "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/MacOS/"

# Firmware
sudo cp hp1020.acl "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/Resources/"

# Fix permissions
sudo chmod 755 "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/MacOS/rasterToHPZJS"
sudo chmod 755 "/Library/Printers/hp/laserjet/hplaserjetzjs.bundle/Contents/MacOS/commandToHPZJS"
```

### 3. Connect the printer

- Plug in the HP LaserJet 1020 via USB (USB-B cable; use a USB-C adapter)
- Power it on
- Confirm macOS sees it:

```bash
lpinfo -v | grep -i usb
# expected output: direct usb://Hewlett-Packard/HP%20LaserJet%201020?serial=...
```

### 4. Add the printer

Copy the full URI from the output above and run:

```bash
lpadmin -p "HP_LaserJet_1020" -E \
  -v "usb://Hewlett-Packard/HP%20LaserJet%201020?serial=YOURSERIAL" \
  -P "/Library/Printers/PPDs/Contents/Resources/HP LaserJet 1022.gz" \
  -D "HP LaserJet 1020"
```

---

## Print

```bash
# print a file
lp -d HP_LaserJet_1020 /path/to/file.pdf

# print a specific page
lp -d HP_LaserJet_1020 -o page-ranges=2 /path/to/file.pdf

# check print queue
lpstat -p HP_LaserJet_1020
```

---

## Notes

- The `lpadmin: Printer drivers are deprecated` warning from CUPS is harmless — printing works fine
- Uses the **LaserJet 1022** PPD, which is fully compatible with the 1020 hardware
- The binaries are universal (x86_64 + arm64) — works on both Intel and Apple Silicon Macs

---

## Disclaimer

This project was made for the internet community to keep old, perfectly functional hardware working on modern operating systems.

This repo is not affiliated with, endorsed by, or in association with HP Inc. in any way. All rights to the original driver files, firmware, and proprietary technology belong to HP Inc. and their respective owners. The files included here are extracted from Apple's publicly distributed HP Printer Drivers package (available via Apple Software Update) solely for compatibility and preservation purposes. No copyright infringement is intended.
