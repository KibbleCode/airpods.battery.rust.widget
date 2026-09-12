# AirPods Battery Widget for Plasma 6, Rust Edition

This is a KDE Plasma 6 remake of the AirPods Battery Widget. Bluetooth scanning,
Apple advertisement validation, battery decoding, device caching, and the D-Bus
service are implemented in Rust. QML remains only for the Plasma applet and its
configuration interface, because Plasma widgets are QML packages.

The project does not use Python, `pip`, a virtual environment, systemd, or a
polling shell process. A session D-Bus activation file starts the Rust backend
when Plasma first requests battery data. This works on systemd and non-systemd
distributions, including Artix Linux.

## Features

- Native Rust scanner built on BlueZ via `bluer`
- Native Rust session D-Bus service built with `zbus`
- AirPods 1, 2, 3, AirPods Pro, and AirPods Pro 2 model detection
- Automatic strongest-device selection or selection of a discovered device
- Average or separate left/right panel levels
- Last-known charging-case level with configurable expiry
- Plasma 6 popup with circular battery indicators and charging badges
- Model-aware icon selection and customizable colors, text, and sizing
- Warning and critical KDE battery notifications
- Automatic D-Bus startup without a background service manager

AirPods Max remains excluded because its advertisement format is not compatible
with the earbud decoder. New models with the same packet layout display as
`AirPods` until their model nibble is known.

## Requirements

- KDE Plasma 6
- BlueZ and a working Bluetooth adapter
- Rust and Cargo
- `pkg-config` and the D-Bus development files
- `kpackagetool6`

On Arch or Artix Linux, the build prerequisites are normally available with:

```bash
sudo pacman -S --needed rust pkgconf dbus bluez bluez-utils
```

## Install

From this project directory, run:

```bash
./scripts/install.sh
```

Then enter Plasma edit mode, choose **Add Widgets**, and add **AirPods Battery
(Rust)**. Open or use the AirPods so they emit a Bluetooth advertisement. The
first result can take several seconds.

The installer builds a release binary and installs:

- Backend: `~/.local/libexec/airpods-battery-service`
- Widget: `~/.local/share/plasma/plasmoids/airpods.battery.rust.widget`
- D-Bus activation: `~/.local/share/dbus-1/services/airpods.battery.rust.widget.service`
- Notification metadata: `~/.local/share/knotifications6/airpodsBatteryRustWidget.notifyrc`

Run `./scripts/install.sh` again to build and install an update. If an older
backend process is already active, log out and back in to use the new binary.

## Development

Run the Rust checks with:

```bash
cargo test
cargo clippy --all-targets -- -D warnings
cargo fmt --check
./scripts/smoke-test.sh
```

Use `./scripts/smoke-test.sh --widget` to also open the applet in
`plasmawindowed` for five seconds and exercise the complete Rust-to-QML path.

Inspect the live backend after installation with:

```bash
qdbus6 airpods.battery.rust.widget /airpods/battery/rust/widget airpods.battery.rust.widget.GetState
```

The D-Bus method returns JSON so the QML boundary stays small and stable. The
JSON includes backend status plus all recently seen supported Apple
advertisements, sorted by signal strength.

## Uninstall

```bash
./scripts/uninstall.sh
```

## Attribution And License

This remake is based on Alessandro Abbenante's
`airpods.battery.widget.frontend`, which in turn uses the AirStatus packet
decoding approach. The original artwork is retained. The project is distributed
under GPL-3.0-only; see `LICENSE`.
