# All About T430s

## Table of Contents
#### **[My Setup](#my-setup)**<br>
#### **[My Operating System](#operating-system)**<br>
- **[How To Tripleboot](#how-to-triple-boot)**<br>
- **[Windows 10](#windows-10)**<br>
- **[Arch Linux](#arch-linux)**<br>
- **[Macos High Sierra](#macos-high-sierra)**<br>

#### **[Reference Link](#reference-link)**<br>

## My Setup
- Processor	: core i7-5320M
- RAM		: 4GB Hynix
- Graphic	: Integrated, Intel HD 4000
- Storage	: HDD 500GB
- Monitor	: HD+ 1600x900
- Keyboard	: Island Style, 6 Row with Backlit

## Operating System
I'm currently using 3 operating system in my machine, Windows 10 build 1803, Arch Linux, and Macos High Sierra. To do this, you must use GPT on your harddrive.

### How To Tripleboot
1. Install Windows 10
2. Install Arch Linux
3. Install Macos High Sierra
4. Install Clover
5. Done

### Windows 10
#### How To Install
I don't have to explained it right?

#### Must Installed Apps
- Lenovo Vantage
- Synaptic Driver
- Dolby

#### How to Fix Biometric Fingerprint Login
After Hackintoshing my machine, i'm unable to login with windows hello via fingerprint. To enable it back, follow this step
1. Open Run Dialog (W+r)
2. Type gpedit.msc then press enter
3. On the left pane, Expand Computer Configuration
4. Expand Administrative Templates
5. Expand Windows Components
6. Select Biometrics
7. Double click Allow the us of biometric
8. Choose Enabled, click OK
9. Double click Allow user to log in using biometrics
10. Choose Enable, click OK
11. Restart and try login using fingerprint

### Arch Linux
#### How To Install
I'm using Install from Existing Linux method, because my internet is suck. So, while waiting for the download package, i can still watch my favourites film or reads some ebook.
In this guide, i'm using Ubuntu 18.04 Live USB from my 16Gb USB Flashdrive.
1. Download iso Ubuntu 18.04 from [here](https://www.ubuntu.com/download/desktop)
2. Download archlinux-bootstrap from [here](https://www.archlinux.org/download)
3. Make bootable usb with rufus or your prefered tool
4. Boot from usb flashdrive
5. Open GParted
6. Make partition for root, home, and swap
7. Shrink your windows 10 partition by 100MB, later this space will be used for enlarging ESP
8. Open terminal, enter superuser mode
9. Mount your ESP <br>
`#mount /dev/sdx1 /mnt` replace sdx1 with your ESP
10. Make Backup of it's content <br>
<pre>
# mkdir ~/esp
# rsync -av /mnt/ ~/esp/</pre>
11. Unmount the ESP <br>
`# umount /mnt`
12. Delete and recreate the ESP
<pre>
# gdisk /dev/sdx # replace sdx with disk containing ESP
p (list partitions)
(ensure the ESP is the first partition)
d (delete partition)
1 (select first partition)
n (create partition)
Enter (use default partition number, should be 1)
Enter (use default first sector, should be 2048)
Enter (use default last sector, should be all available space)
EF00 (hex code for EFI system partition)
w (write changes to disk and exit)
</pre>
13. Format the ESP
<pre>
# partprobe /dev/sdx
# mkfs.fat -F32 /dev/sdx1
</pre>
14. Restore ESP's contents
<pre>
# mount /dev/sdx1 /mnt
# rsync -av ~/esp/ /mnt
</pre>
15. Unmount ESP <br>
`# umount /mnt`
16. Move to /tmp directory <br>
`# cd /tmp`
17. Extract the archlinux-bootstrap image <br>
`# tar  xzf <path-to-bootstrap-image>/archlinux-bootstrap-2018.04.01-x86_64.tar.gz`
18. Mount your root partition to /tmp/root.x86_64/mnt <br>
`# mount /dev/sdxX` replace sdxX with yout root partition
19. Make home and boot folder inside /tmp/root.x86_64/mnt
<pre>
# mkdir /tmp/root.x86_64/mnt/{home,boot}
</pre>
20. Mount your home and ESP partition to home and boot. <br> Note: replace sdxY with your home partition and sdxZ with your ESP
<pre>
# mount /dev/sdxY /tmp/root.x86_64/mnt/home
# mount /dev/sdxZ /tmp/root.x86_64/mnt/boot
</pre>
21. Chroot to the extracted directory <br>
`# /tmp/root.x86_64/bin/arch-chroot /tmp/root.x86_64/`
22. Initialize pacman keyring
<pre>
# pacman-key --init
# pacman-key --populate archlinux
</pre>
23. Prepairing mirror
Edit mirror with local mirror for faster download speed, here i'm use Indonesian server <br>
`# nano /etc/pacman.d/mirrorlist` <br>
add this line
<pre>
#poliwangi
Server = http://mirror.poliwangi.ac.id/archlinux/$repo/os/$arch
#ubaya
Server = http://suro.ubaya.ac.id/archlinux/$repo/os/$arch
#kambing
Server = http://kambing.ui.ac.id/archlinux/$repo/os/$arch
</pre>
24. Installing base
<pre>
# pacman -Syy
# pacstrap /mnt base base-devel
</pre>
25. Generate fstab <br>
`genfstab -L -p -P /mnt > /mnt/etc/fstab`
26. Chroot to new arch system <br>
`# arch-chroot /mnt`
27. Setting up system <br>
Make hostname <br>
`# echo "Your-Desired_Hostname" >> /etc/hostname` <br>
Set Locale <br>
`# nano /etc/locale.gen` <br>
Uncomment en_US.UTF-8 UTF-8 dan id_ID.UTF-8 UTF-8, then make locale.conf file <br>
`# nano /etc/locale.conf` <br>
Add this line
<pre>
LC_COLLATE=C
LANG=en_US.UTF-8
LC_TIME=id_ID.UTF-8
</pre>
Generate local with `# locale-gen` command <br>
Create symbolic link zone <br>
`# 	
ln -sf /usr/share/zoneinfo/Asia/Jakarta /usr/localtime`
28. Installing some packages
<pre>
# pacman -S bash-completion
# pacman -S ntfs-3g wpa_supplicant dialog
# pacman -S xorg xorg-xinit xf86-video-intel openbox obconf lxappearance
</pre>
29. Setting up new user <br>
Make sudo group <br>
`# groupadd sudo` <br>
Add new user and assign to sudo group <br>
`# useradd  -m -g users -G sudo,power,storage your-user-name` <br>
Edit sudoers file  <br>
`# nano /etc/sudoers` <br>
Add or uncomment this line <br>
`%sudo ALL=(ALL)` <br>
make password for your user and root
<pre>
# passwd your-user-name
# passwd root
</pre>
30. Install bootloader <br>
`# mkinitcpio -p linux`<br>
For Intel processor user, install intel-ucode <br>
`# pacman -S intel-ucode` <br>
Install systemd-boot <br>
`# bootctl install` <br>
Write systemd-boot entry in /boot/loader/entries <br>
`# nano /boot/loader/entries/archlinux.conf` <br>
Add this lines
<pre>
title Archlinux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=/dev/sdaX rw
</pre>
31. Reboot
32. Setting up display manager and wm <br>
Login with your username <br>
Create a copy of the default xinitrc in your home directory <br>
`$ cp /etc/X11/xinit/xinitrc ~/.xinitrc` <br>
Edit .xinitrc <br>
`$ nano .xinitrc` <br>
Add this line
<pre>
exec openbox
</pre>
Save file and run startx <br>
`$ startx`

### Macos High Sierra
#### How To Install

### Reference Link
- https://superuser.com/questions/1230741/how-to-resize-the-efi-system-partition
- https://wiki.archlinux.org/index.php/Install_from_existing_Linux
- https://windowsreport.com/fix-windows-hello-fingerprint-wont-work/