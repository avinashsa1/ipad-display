# iPad as a wireless extended display for your Fedora laptop

Turns your iPad into a **true extended monitor** over Wi-Fi — not screen mirroring.
You drag windows from your laptop screen onto the iPad, just like a plugged-in display.

## How it works

1. **`ipad-display start`** creates a *virtual monitor* inside your current desktop session
   (X11 → `xrandr` VIRTUAL output · GNOME/Wayland → mutter's D-Bus API). The desktop
   treats it as a real second screen.
2. **Sunshine** (self-hosted, open source) streams *only that monitor* with low-latency
   GPU encoding (H.264/HEVC).
3. **Moonlight** (free on the App Store) on your iPad receives it. Touch works as a mouse.

## Quick start

```bash
cd ~/workspace/ipad-extended-display
chmod +x ipad-display

# 1. Install Sunshine + helpers, open firewall ports (one time, needs sudo)
./ipad-display install

# 2. Create the virtual display and start streaming
./ipad-display start

# 3. Follow the printed pairing steps on your iPad, then:
./ipad-display stop    # turn the virtual monitor back off when done
./ipad-display status  # diagnose anything
```

### Pairing cheat-sheet (also printed by `./ipad-display pair`)

- **Laptop:** open `https://localhost:47990`, create a Sunshine username + password.
- **iPad:** install **Moonlight** (free, App Store), join the **same Wi-Fi** as the laptop.
- In Moonlight tap **+**, enter your laptop's IP (printed by the script), note the 4-digit PIN.
- Back in Sunshine's web UI, type the PIN → **Send**. Tap the laptop in Moonlight. Done.

## Options

```bash
./ipad-display start --res 2360x1640        # match your iPad's resolution
./ipad-display start --pos left             # put the virtual screen left of the laptop
./ipad-display start --monitor-only         # create the monitor, don't stream
./ipad-display start --output 2             # override which display Sunshine streams
```

### Resolution guide

Default is `1920x1440` (4:3 — fills an iPad nicely). Native iPad resolutions if you want
pixel-perfect sharpness:

| iPad | Native resolution |
|---|---|
| iPad Pro 11" (M4) | 2420x1668 |
| iPad Air 11" / Pro 11" (older) | 2360x1640 |
| iPad 10.9" / Air (older 10.9") | 2360x1640 |
| iPad Pro 12.9" / 13" | 2732x2048 |
| iPad mini | 2266x1488 |

Higher resolution = sharper but more Wi-Fi bandwidth. `1920x1440` is the sweet spot on
most home networks.

## Requirements

- Fedora laptop (40+), GNOME desktop (Wayland or X11 session)
- iPad with the free **Moonlight** app
- Both devices on the **same Wi-Fi** (5 GHz strongly recommended)
- X11 session additionally needs an Intel iGPU (for `VIRTUAL` outputs); on AMD/NVIDIA
  under X11, log into a GNOME/Wayland session instead

## Tips

- **Moonlight settings:** 60 FPS, HEVC/H.265 if offered, and "Stretch video to fill" off
  if you matched the iPad's native resolution.
- **Input:** single-finger tap = click, drag = move, two-finger tap = right-click.
  Apple Pencil works as a precise mouse.
- **Power:** keep the laptop plugged in and disable auto-suspend while streaming
  (Settings → Power).
- **Audio:** Moonlight can also forward laptop audio to the iPad — toggle it in
  Moonlight's settings if you want it.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Moonlight can't find the laptop | Same Wi-Fi? Check `./ipad-display status` IPs. Re-run `install` for firewall ports. |
| Sunshine shows the *laptop* screen, not the virtual one | The script sets `output_name` automatically; verify in `~/.config/sunshine/sunshine.conf`, or pick the display in the web UI and restart Sunshine. |
| Black screen in Moonlight (Wayland) | First run needs a screen-share permission prompt on the laptop — approve it in the GNOME dialog. |
| PIN box missing in Sunshine web UI | Create the web-UI username/password first at `https://localhost:47990`. |
| `start` fails on X11: no VIRTUAL output | Your GPU driver doesn't expose virtual outputs — use a GNOME/Wayland session instead. |
| Laggy | Switch to 5 GHz Wi-Fi, drop to `--res 1920x1440`, 60→30 FPS in Moonlight. |
| Virtual monitor stuck after `stop` (Wayland) | It clears on logout; or disable it in Settings → Displays. |

## Uninstall

```bash
./ipad-display stop
sudo dnf remove Sunshine
sudo dnf copr disable lizardbyte/stable
sudo firewall-cmd --permanent --remove-port=47984/tcp --remove-port=47989/tcp \
  --remove-port=47990/tcp --remove-port=48010/tcp --remove-port=47998-48010/udp
sudo firewall-cmd --reload
rm -rf ~/.config/sunshine
```

## Security notes

- Sunshine/Moonlight pairing uses a PIN + TLS; the stream never leaves your LAN.
- Firewall rules only open Sunshine's ports (47984, 47989, 47990, 48010 TCP; 47998–48010 UDP).
- Don't expose port 47990 to the internet — it's your admin console.

## Files

- `ipad-display` — the whole tool (bash, no dependencies beyond what `install` sets up)
- `README.md` — this guide
