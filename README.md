# RaspbPiConfig
Personal config for raspberry pi 3B+

# SSH at start
Create a file named `ssh` in root of the SD
```
cd a sdcard
type NUL >> ssh
```

# Wifi at start
create `wpa_supplicant.conf` at root of SD:
```
country=AR
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="NETWORK-NAME"
    psk="NETWORK-PASSWORD"
}
```

# First boot
```
sudo raspi-config
```
change hsotname and expand filesystem

# Create user
```
sudo adduser USERHERE
for GROUP in adm dialout cdrom sudo audio video plugdev games users netdev input spi i2c gpio; do sudo adduser USERHERE $GROUP; done
```

# Autologin to CLI

Edit your /etc/systemd/logind.conf , change #NAutoVTs=6 to NAutoVTs=1

Create a /etc/systemd/system/getty@tty1.service.d/override.conf through ;

systemctl edit getty@tty1

Past the following lines
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin USERHERE --noclear %I 38400 linux
```
enable the getty@tty1.service then reboot

systemctl enable getty@tty1.service

reboot

# Install ohmyZSH
`sudo apt install zsh`


`sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`
