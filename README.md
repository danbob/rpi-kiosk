# Raspberry Pi kiosk with idle screen blanking

This repo has the basic configuration you need to turn a touchscreen-equipped Raspberry Pi into a kiosk served from a webpage. It also features screen blanking when idle / touch to wake (this is more difficult to get right than it sounds).

Tested on a Raspberry Pi 3 B+ running Raspbian Stretch.

## How to use

Install needed packages:

```bash
sudo apt install --no-install-recommends \
  xserver-xorg \
  x11-xserver-utils \
  xinit \
  wmctrl \
  suckless-tools \
  openbox \
  chromium-browser
```

Copy the files to the home directory of the non-root user you wish to use:

```bash
cp -r rpi-kiosk/{.config,.xserverrc,sleep-timer} ~

tail -n 1 rpi-kiosk/.profile-snippet >> .profile

cp /etc/X11/xinit/xinitrc .xinitrc
tail -n 1 rpi-kiosk/.xinitrc-snippet >> .xinitrc

chmod +x ~/sleep-timer
```

Enable console autologin with the `raspi-config` tool (Boot Options > Desktop / CLI > Console Autologin). If you're setting this up from root for a user without sudo, this method won't work. Instead, run the script noninteractively:

```bash
SUDO_USER=some_user raspi-config nonint do_boot_behavior B2
```

If you're using a new user and not the default `pi`, add it to the `video` group:

```bash
sudo usermod -a -G video some_user
```

## How it works

The configuration in `.xinitrc`, `.xserverrc`, `.profile`, and `autostart` are all that you need for a simple kiosk. If you want the screen to turn off when idle, however, you have to get a bit clever. Touching the screen to wake it also triggers a blind click event which you probably don't want. To avoid this, I disable automatic screen blanking and control it manually with the `sleep-timer` script. Just before turning off the screen, the script switches to a second (empty) openbox desktop. In `rc.xml` I've disabled the desktop switching indicator and added a mouse bind that responds to a click event on that empty desktop by switching back to the first one.

## Credits

Some resources that helped me put all this together:
- https://die-antwort.eu/techblog/2017-12-setup-raspberry-pi-for-kiosk-mode/
- http://openbox.org/wiki/Help:Contents
- https://superuser.com/a/1094239
