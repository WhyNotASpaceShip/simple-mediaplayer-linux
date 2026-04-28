# SimplePlayer

A single-file, zero-dependency HTTP movie player for Linux.  
Serves a local folder of videos and lets you browse + play them in any browser.

## Requirements

- Python 3.6+  (already on Rocky Linux 10)
- A modern browser (Chrome, Firefox, …)
- **ffmpeg** *(optional but recommended)* — enables audio for MKV and other formats

## Usage

```bash
# serve the current directory on port 8080 (default)
python3 player.py

# serve a specific folder
python3 player.py /path/to/movies

# specify folder + port
python3 player.py /path/to/movies 9000
```

Then open http://localhost:8080 (or the network URL printed in the terminal).

## Supported formats

`.mp4` `.m4v` `.mkv` `.webm` `.avi` `.mov` `.ts` `.ogv` `.ogg` `.wmv` `.mpg` `.mpeg`

> Browser playback depends on codec support.  
> **mp4/h264** and **webm/vp9** work natively in all modern browsers.

### Why MKV (and AVI/WMV) may have no audio without ffmpeg

Browsers only decode a small set of audio codecs: **AAC, MP3, Opus, Vorbis**.  
MKV files commonly carry **AC3, EAC3, or DTS** audio — codecs browsers refuse to play.  
That is why an MKV with AC3 audio looks fine but plays silently.

When **ffmpeg** is installed, SimplePlayer automatically transcodes these formats on-the-fly:
the video stream is passed through untouched, and only the audio is re-encoded to AAC so the
browser can play it. MP4 files use AAC natively and are never transcoded.

> **Note:** Transcoded streams (MKV, AVI, etc.) do not support seeking to an arbitrary position
> by dragging the progress bar. MP4 files are unaffected and remain fully seekable.

### Installing ffmpeg

```bash
# Rocky Linux / RHEL (requires EPEL + RPM Fusion)
sudo dnf install epel-release
sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm
sudo dnf install ffmpeg

# Fedora
sudo dnf install ffmpeg

# Ubuntu / Debian
sudo apt install ffmpeg
```

The startup banner will confirm whether ffmpeg was found:

```
  ffmpeg  : /usr/bin/ffmpeg (MKV/AVI audio transcoding enabled)
```

## Keyboard shortcuts (in player)

| Key | Action |
|-----|--------|
| `Space` | Pause / Play |
| `← / →` | Seek −10 s / +10 s |
| `↑ / ↓` | Volume −10% / +10% |
| `N / P` | Next / Previous video |
| `F` | Fullscreen |

## Run as a systemd service (optional)

```ini
# /etc/systemd/system/simpleplayer.service
[Unit]
Description=SimplePlayer
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/simpleplayer/player.py /path/to/movies 8080
Restart=on-failure
User=nobody

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now simpleplayer
```

## Firewall (Rocky Linux)

```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```
