## Fedberry Installation

Copy minimal image to sd card:

```bash
xzcat fedberry-minimal-28.0.20180725.raw.xz | sudo dd of=/dev/mmcblk0 bs=4M iflag=fullblock oflag=direct status=progress; sudo sync
```

Modify **/config.txt** to disable ethernet enerfy efficiency by put the line below to the end of the file:

```bash
dtparam=eee=off
```

Modify **/cmdline.txt** to disable IPv6 by add **ipv6.disable=1** to the line

After installing minimal image we need to do basic settings in **fedberry-config**, for example disabling SELinux; enable i2c, spi, watchdog and RTC:

```bash
sudo fedberry-config
```

After we reboot the system, do an update and reboot again:

```bash
sudo dnf upgrade --refresh
sudo shutdown -r now
```

We want to be an administrator, enable and start sshd:

```bash
sudo su -

systemctl enable sshd
systemctl start sshd
```

Enlarge swap file to 2GB:

```bash
swapoff /swapfile
rm /swapfile
pv -p --size 2048M -S /dev/zero | dd of=/swapfile bs=1k count=2048k status=noxfer
mkswap /swapfile
chmod 600 /swapfile
swapon /swapfile
```

Put /temp to physical file system by adding a pendrive:

```bash
nano /etc/fstab

---
#tmpfs /tmp tmpfs    defaults,noatime,size=100m 0 0
/dev/sda1 /tmp ext4    defaults,noatime 0 0
---
```

Installing hungarian locale:

```bash
dnf install glibc-langpack-hu
```

Set hungarian locale:

```bash
localectl set-locale LANG=hu_HU.UTF-8
localectl set-keymap hu
localectl set-x11-keymap hu
```

Setting up hostname:

```bash
hostnamectl set-hostname <hostname>
```

*Optional - Configure network settings:*

```bash
nmcli
systemctl restart network
```

Install development tools:

```bash
dnf group install "Development Tools" "C Development Tools and Libraries"
dnf install mc gcc-c++ gdb gdb-gdbserver htop ftop iotop cmake p7zip p7zip-plugins ntop
dnf install clang lldb python2-lldb lld libcxx
```

Install PostgreSQL and initialize DB:

```bash
dnf install postgresql postgresql-server postgresql-contrib postgresql-devel
postgresql-setup --initdb --unit postgresql
```

Setting up MD5 authentication ion IPv4 and IPv6 rows:

```bash
nano /var/lib/pgsql/data/pg_hba.conf
```

We need to start PostgreSQL server and create password for **postgres** (admin) user:

```bash
systemctl enable postgresql
systemctl start postgresql

sudo -u postgres psql postgres
ALTER USER postgres WITH PASSWORD 'enigma';
\q
```

Install development libraries mainly for Qt5, OpenCV and SDL2:

```bash
dnf install fontconfig-devel dbus-devel libicu-devel libinput-devel libxkbcommon-devel sqlite-devel openssl-devel libssh-devel libpng-devel libjpeg-turbo-devel glib2-devel
dnf install bluez-libs-devel alsa-lib-devel pulseaudio-libs-devel cups-devel fftw-devel boost-devel
dnf install gstreamer1-devel gstreamer1-plugins-base-tools gstreamer1-plugins-base-devel gstreamer1-plugins-good gstreamer1-plugins-good-extras gstreamer1-plugins-ugly gstreamer1-plugins-bad-free gstreamer1-plugins-bad-free-devel gstreamer1-plugins-bad-free-extras
dnf install jasper-devel libtiff-devel atlas-devel eigen3-devel tbb-devel
dnf install freeimage-devel openal-soft-devel pango-devel libsndfile-devel systemd-devel libwebp-devel libmodplug-devel freeglut-devel
dnf install libXrandr-devel libXcursor-devel libXi-devel libXinerama-devel libXScrnSaver-devel
```

Install Raspberry Pi-specific libs:

```bash
dnf install raspberrypi-vc-utils raspberrypi-vc-libs-devel raspberrypi-vc-static omxplayer wiringpi-devel wiringpi-tools
```

Reboot the system, and install Rust:

```bash
shutdown -r now

curl https://sh.rustup.rs -sSf | sh

rustup self update
rustup update

rustup component add rls-preview rust-analysis rust-src rustfmt-preview
cargo install sccache racer cargo-update cargo-make rustsym
```

*Optional - install LXDE*

```bash
sudo su -c 'dnf group install lxde-desktop base-x'
sudo dnf install xterm xclock
```

*Disable dnfdragora*

```bash
sudo mv /etc/xdg/autostart/org.mageia.dnfdragora-updater.desktop /etc/xdg/autostart/org.mageia.dnfdragora-updater.desktop.mask
```


*Setting up graphical login as default*

```bash
sudo systemctl set-default graphical.target
```

*Get back to text login:*

```bash
sudo systemctl set-default multi-user.target
```

*To check the current target:*

```bash
sudo systemctl get-default
```

*Optional - Building Visual Studio Code*

```bash
sudo dnf install nodejs libsecret-devel libxkbfile-devel GConf2-devel
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo dnf install yarn
```

*Getting VS Code sources*

```bash
git clone https://github.com/microsoft/vscode
```

*Building VS Code*

```bash
cd vscode
./scripts/npm.sh install --arch=armhf
```

*Adding Extensio Gallery to product.json*

```bash
"extensionsGallery": {
    "serviceUrl": "https://marketplace.visualstudio.com/_apis/public/gallery",
    "cacheUrl": "https://vscode.blob.core.windows.net/gallery/index",
    "itemUrl": "https://marketplace.visualstudio.com/items"
}
```

*Starting the IDE*

```bash
./scripts/code.sh
```

Backing up the whole system:

```bash
sudo dd if=/dev/mmcblk0 bs=4M iflag=fullblock status=progress | bzip2 > fedberry-28.img.bz2
```
