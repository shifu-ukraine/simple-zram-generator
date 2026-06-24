# simple-zram-generator

A flexible, configuration-driven Bash script that automatically generates native `systemd` units, `udev` rules, and `modprobe` options based on a simple layout configuration file. Ideal for custom, high-performance setups on rolling-release distributions like **Gentoo** or **Arch Linux**.

## Features

- **Single-pass parsing:** Efficiently processes configuration and automatically counts devices in one single go.
- **Dynamic Module Options:** Calculates and writes the correct `num_devices` value for the `zram` kernel module dynamically.
- **Native systemd approach:** Generates individual `.swap`, `.mount`, and companion formatting `.service` files with bulletproof dependency mapping (`After=`, `Requires=`, `Before=`).
- **No external block-device wrappers:** Uses systemd's native declarative syntax, eliminating unnecessary background daemons or messy wrapper scripts.

## File Layout

1. `/etc/zram-slices.conf` — The layout configuration file where you define your ZRAM devices.
2. `simple-zram-generator` — The deployment script (must be run as root).

## Configuration Syntax

Define your ZRAM slices in `/etc/zram-slices.conf` using the following pipe-separated format:
`"SIZE | COMPRESSION_ALGORITHM | TYPE (swap|tmp|mount) | TARGET_PATH_OR_LABEL"`

### Example `/etc/zram-slices.conf`

```bash
# Format: SIZE | COMP_ALGO | TYPE | TARGET_OR_LABEL
"16G | zstd | swap  | zswap"
"1G  | zstd | tmp   | /tmp"
"22G | zstd | mount | /var/tmp/portage"
```

## Installation & Deployment

1. Clone this repository or copy the script to your machine.
2. Create your configuration file at `/etc/zram-slices.conf` using the syntax above.
3. Run the generator script with root privileges:

```bash
sudo chmod +x simple-zram-generator
sudo ./simple-zram-generator
```
4. **Important:** If your system uses an `initramfs` (e.g., generated via `dracut` or `mkinitcpio`), you **must rebuild it** now. This ensures that the new `modprobe` and `modules-load` configurations are properly baked into the early boot stage.
5. Reboot your system to cleanly apply the new kernel module options, trigger udev rules, and mount everything natively.

## How It Works Under the Hood

The script reads your layout and populates the system with these native configurations:
- `/etc/modules-load.d/zram.conf` — forces the `zram` module to load at boot.
- `/etc/modprobe.d/zram.conf` — configures the exact `num_devices` dynamically based on your config.
- `/etc/udev/rules.d/10-zram.rules` — sets disk sizes and compression algorithms as soon as device nodes appear.
- Custom systemd units under `/etc/systemd/system/` (including standard drop-in overrides for `tmp.mount` if `/tmp` is targeted).

## License
[MIT License](LICENSE)
