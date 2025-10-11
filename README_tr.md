# QuemOS Live ISO Kurulum ve Derleme Talimatları

Bu dosya QuemOS (Debian tabanlı) bir sistemin sıfırdan nasıl derleneceğini anlatır.
Tüm işlemler root yetkisiyle yapılmalıdır.


## 1. Gerekli paketleri ana makinenize kurun
```
apt-get install debootstrap xorriso squashfs-tools mtools grub-pc-bin grub-efi-ia32-bin grub-efi
```


## 2. Boş Debian kök sistemi oluşturun

```
mkdir quemos-chroot
```

```
debootstrap --arch=amd64 --no-merged-usr stable quemos-chroot https://deb.debian.org/debian
chown root quemos-chroot
```

```
echo "APT::Sandbox::User root;" > quemos-chroot/etc/apt/apt.conf.d/99sandboxroot
```

## 3. Chroot ortamına girin ve kaynakları ayarlayın

```
chroot quemos-chroot /bin/bash
```

```
echo 'deb http://deb.debian.org/debian trixie main contrib non-free non-free-firmware' > /etc/apt/sources.list
```

```
echo 'deb http://deb.debian.org/debian-security trixie-security main contrib non-free non-free-firmware' >> /etc/apt/sources.list
```

```
echo 'deb http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware' >> /etc/apt/sources.list
```

```
echo "deb [trusted=yes] https://quemos.github.io/repo stable main" | tee /etc/apt/sources.list.d/quemos.list 
```

```
apt-get update
```


## 4. Temel sistem ve GNOME masaüstünü kurun

```
apt-get install linux-headers-amd64 linux-image-amd64 grub-pc-bin grub-efi grub-efi-ia32-bin ssh sudo htop fastfetch wget git curl unzip zip tar xz-utils gnupg ca-certificates vlc transmission-gtk gnome-core gdm3 gnome-tweaks gnome-software gnome-software-plugin-flatpak network-manager network-manager-gnome pipewire wireplumber pavucontrol evince eog gnome-boxes gvfs-backends fonts-noto fonts-dejavu libreoffice flatpak gdebi remmina remmina-plugin-rdp remmina-plugin-vnc remmina-plugin-spice extrepo live-boot live-config wine winetricks gnome-shell-extensions gnome-shell-extension-prefs chrome-gnome-shell bleachbit gvfs gvfs-backends gvfs-fuse -y && curl -fsS https://dl.brave.com/install.sh | sh
```

## 5. Flatpak’i etkinleştirin

```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## 6. Waydroid kurun

```
curl -s https://repo.waydro.id | bash -s trixie
```

```
apt install waydroid -y
```

## 7. Donanım firmware paketlerini yükleyin (isteğe bağlı)

```
apt-get install bluez-firmware firmware-amd-graphics firmware-atheros \
  firmware-b43-installer firmware-b43legacy-installer firmware-bnx2 \
  firmware-bnx2x firmware-brcm80211 firmware-cavium firmware-intel-sound \
  firmware-intelwimax firmware-ipw2x00 firmware-ivtv firmware-iwlwifi \
  firmware-libertas firmware-linux firmware-linux-free firmware-linux-nonfree \
  firmware-misc-nonfree firmware-myricom firmware-netxen firmware-qlogic \
  firmware-ralink firmware-realtek firmware-samsung firmware-siano \
  firmware-ti-connectivity firmware-zd1211 zd1211-firmware
```

## 8. Live Installer paketini yükleyin

```
cd /tmp/
```

```
wget https://github.com/quemos/deb/releases/download/deb/17g-installer_1.0_all.deb
```

```
dpkg -i /tmp/17g-installer_1.0_all.deb
```

```
apt-get install -f -y
```

## 9. Chroot’tan çıkın ve bağlantıları kaldırın

```
exit
```

```
umount -lf -R quemos-chroot/* 2>/dev/null
```

## 10. Temizlik işlemleri

```
chroot quemos-chroot apt-get autoremove
```

```
chroot quemos-chroot apt-get clean
```

```
rm -f quemos-chroot/root/.bash_history
```

```
rm -rf quemos-chroot/var/lib/apt/lists/*
```

```
find quemos-chroot/var/log/ -type f | xargs rm -f
```


## 11. ISO dosyası için yapılandırma klasörü oluşturun

```
mkdir isowork
```

```
mksquashfs quemos-chroot filesystem.squashfs -comp gzip -wildcards
```

```
mkdir -p isowork/live
```

```
mv filesystem.squashfs isowork/live/filesystem.squashfs
```

## 12. Kernel ve initrd dosyalarını kopyalayın

```
ls quemos-chroot/boot/
```

```
cp -pf quemos-chroot/boot/initrd.img-5.7.0-1-amd64 isowork/live/initrd.img
```

```
cp -pf quemos-chroot/boot/vmlinuz-5.7.0-1-amd64 isowork/live/vmlinuz
```

## 13. GRUB yapılandırmasını oluşturun

```
mkdir -p isowork/boot/grub/
```

```
echo 'insmod all_video' > isowork/boot/grub/grub.cfg
```

```
echo 'menuentry "Start QuemOS Live" --class debian {' >> isowork/boot/grub/grub.cfg
```

```
echo '    linux /live/vmlinuz boot=live live-config live-media-path=/live --' >> isowork/boot/grub/grub.cfg
```

```
echo '    initrd /live/initrd.img' >> isowork/boot/grub/grub.cfg
```

```
echo '}' >> isowork/boot/grub/grub.cfg
```

## 14. ISO dosyasını oluşturun

```
grub-mkrescue isowork -o quemos-live.iso
```


Notlar:
-------
- Bu işlemler sonunda “quemos-live.iso” adlı canlı sistem oluşturulur.
- İmaj, UEFI ve BIOS desteklidir.
- Eğer farklı tema, kullanıcı adı veya yapılandırma isteniyorsa bunlar chroot içindeyken yapılmalıdır.
