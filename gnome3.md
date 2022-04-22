# GNOME 3 tweaks

## Speed up (web) scrolling

This arranges for faster mouse wheel scrolling speed in certain web browsers (notably Web and Chromium)
where, unlike in Firefox, the scrolling is painfully slow.

### Install the [imwheel](http://imwheel.sourceforge.net/) package

#### Arch Linux

```
sudo pacman -S imwheel
```

### Configure imwheel

`~/.imwheelrc`:

```
"^epiphany$"
None, Up,   Button4, 4
None, Down, Button5, 4

"^chromium$"
None, Up,   Button4, 4
None, Down, Button5, 4
```

https://en.linuxportal.info/tutorials/graphical-user-interface/general/how-to-control-the-mouse-gorget-speed-with-the-help-of-the-imwheel-program

### Configure systemd unit for imwheel

`~/.config/systemd/user/imwheel.service`:

```
[Unit]
Description=IMWheel
Wants=display-manager.service
After=display-manager.service

[Service]
Type=simple
Environment=XAUTHORITY=%h/.Xauthority
ExecStart=/usr/bin/imwheel -d
ExecStop=/usr/bin/pkill imwheel
RemainAfterExit=yes

[Install]
WantedBy=graphical-session.target
```

### Enable the systemd unit

```
sudo systemctl daemon-reload
systemctl --user enable imwheel
systemctl --user start imwheel
```

https://wiki.archlinux.org/title/Systemd/User