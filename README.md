**1. Задача - Попасть в систему без паролā несколькими способами**

Первый способ:

Нажал **e** в меню выбора прогарммы GRUB.

В конце строки с параметрами запуска ядра, которая начинается с linux16, добавил параметр **init=/bin/sh** и нажал Ctrl-X для загрпузки.

Такми способом попал в систему без пароля. При этому файловая система смонтировалась в режиме ro. Для монтирования в режим rw использовал компанду:

`mount -o remount,rw /`

Второй способ:

В той же строке параметров запуска ядра добавил параметр **rd.break**.

В этом режиме поменял пароль администратора командами (заранее смонтировав файловую систмесу рута в режим rw `mount -o remount,rw /sysroot`):

Изменил корневую файловую систему

`chroot /sysroot`

Изменин пароля рута

`passwd root`

`touch /.autorelabel`

Перемонтировал обратно в режим ro

`mount -o remount,ro /`

И проверил последний третий вариант:

Заменил параметр **ro** на **rw init=/sysroot/bin/sh** 

Последним вариантом файловая система смонтировалась в режиме rw.

___

**2. Задача - Установить систему с LVM, после чего переименовать VG**

Проверил текущее состояние системы

`sudo vgs`

```
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
```

Переименовал Volume Group

`vgrename VolGroup00 OtusRoot`

```
 Volume group "VolGroup00" successfully renamed to "OtusRoot"
```

Проверил

`usod vgs`

```
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0
```

Переименовал VolGroup00 на OtusRoot в файлах  /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg.

Пересоздал initrd image чтобý он знал новое имя VG командой по рутом

`mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)`

```
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** Constructing AuthenticAMD.bin ****
*** Store current command line parameters ***
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```

После чего перезагрузился и проверил.

`reboot`

`sudo vgs`

```
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0
```
___

**3. Задача - Добавить модуль в initrd**

Для создание своего модуля добавил папку с модулями

`[root@lvm ~]# mkdir /usr/lib/dracut/modules.d/01test`

В созданную директорию поместил два скрипта

[module_setup.sh](https://github.com/Sergey-SSA/boot/blob/master/gistfile1.txt) и [test.sh](https://github.com/Sergey-SSA/boot/blob/master/test.sh)

```
[root@lvm 01test]# cat test.sh
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
[root@lvm 01test]# cat
module_setup.sh  test.sh
[root@lvm 01test]# cat module_setup.sh
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
[root@lvm 01test]# ls -la
total 12
drwxr-xr-x.  2 root root   44 May 23 12:54 .
drwxr-xr-x. 52 root root 4096 May 23 12:52 ..
-rw-r--r--.  1 root root  126 May 23 12:53 module_setup.sh
-rw-r--r--.  1 root root  332 May 23 12:54 test.sh
[root@lvm 01test]# pwd
/usr/lib/dracut/modules.d/01test
```

Пересобрал образ initrd командами

`dracut -f -v`

Далее удалил опции rghb и quiet в файле grub.cfg

Для проверки перезапустил ВМ и после чего в процессе загрузки появлялся пингвин.
