# Temu T7 Tablet – Notes, Reverse Engineering & Spoof Busters

**Link to Tablet:** [Temu T7](https://share.temu.com/9a9lbxwkwSC)

---

## 📦 First Impressions
Upon arrival, the T7 doesn’t exactly scream “premium.”  
- It’s light, plasticky, and about as durable as a Pringles can.  
- Boot time? Think Windows Vista with 3 years of uninstalled updates.    

Naturally, this only made me more curious. Time to crack it open (figuratively, not literally… yet).

---

## 🛠 Step 1: Developer Mode Shenanigans
Enable Developer Mode → Enable ADB → (Optional) Enable OEM Unlock.

First command:
`adb root on`


Response:

>...cannot run root in production builds


BUMMERRRRR! (but hey, expected.)

---

## 🧠 Step 2: CPU Reality Check
`cat /proc/cpuinfo`

```
T7:/ $ cat /proc/cpuinfo
Processor	: AArch64 Processor rev 4 (aarch64)
processor	: 0
model name	: AArch64 Processor rev 4 (aarch64)
BogoMIPS	: 26.00
BogoMIPS	: 26.00
Features	: fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer	: 0x41
CPU architecture: 8
CPU variant	: 0x0
CPU part	: 0xd03
CPU revision	: 4

Hardware	: MT6735P
```

###
Result:
SoC: MT6735P

Cores: Cortex-A53 (2015-era tech)

So yeah, we’re rocking silicon old enough to remember when Pokémon Go was new.\
Spoiler: this SoC and Android 14 go together about as well as dial-up and Netflix.

--- 

## 🔍 Step 3: System Property Spelunking
`getprop`

And here’s where the clown shoes showed up:

This is not a spec sheet.  
This is a **fanfiction** of what the T7 *wishes* it could be.  

- Snapdragon 865? → Nah, try MT6735P.  
- 16 GB RAM? → More like 2 GB and a prayer.  
- 512 GB storage? → Porbably more like 16 GB eMMC saying *"I’m doing my best, bro."*  
- 10 CPU cores? → If you count the *marketing department*.  
- 8800 mAh battery? → Maybe if you daisy-chain a couple of power banks.  

Honestly, it reads like a Craigslist ad written by someone who thinks “cores” are a breakfast cereal. 😂

```
[persist.sys.dalvik.vm.lib.2]: [libart.so]
[persist.sys.device.model]: [T7]
[persist.sys.fake.androidver]: [14]
[persist.sys.fake.battery_cap]: [8800]
[persist.sys.fake.camera_back]: [3200]
[persist.sys.fake.camera_front]: [2400]
[persist.sys.fake.cpu]: [Snapdragon 865]
[persist.sys.fake.cpu_cores]: [10]
[persist.sys.fake.ram]: [16]
[persist.sys.fake.resolution]: [2560x1600]
[persist.sys.fake.screen_size]: [11.6]
[persist.sys.fake.storage]: [512]
[ro.serialno]: [0123456789ABCDEF] ← Bruh. LMFAO.
```

Reality check:

```
[ro.build.version.sdk]: [24] → That’s Android 7 (Nougat).
[ro].build.version.release]: [14.0] → Pure cosplay.
```

So this “Android 14 tablet” is basically wearing a Nougat onesie with a fake mustache.

---

## 💾 Step 4: Memory Reality Check
Advertised RAM: 16 GB (lol).
Actual RAM:

```
free -h
total        used        free
Mem:   1.3G        1.3G         80M
Swap:   707M         24M        683M
```

Translation: ~2 GB real RAM, duct-taped with swap.  
Yeahhh… unless wishful thinking counts as memory, this ain’t 16 GB.  

---

## 📂 Step 5: Partition Safari

Good news: SELinux is Permissive → perfect for tinkering.  
Bad news: no scatter file included.  
Solution: I made my own. 💪  

***
How:

Listed partitions via `/dev/block/platform/.../by-name/.`  
Exact command: `ls -la /dev/block/platform/mtk-msdc.0/11230000.msdc0/`  

Results? 

```
T7:/ $ ls -la /dev/block/platform/mtk-msdc.0/11230000.msdc0/by-name/                                                         
total 0
drwxr-xr-x 2 root root 500 2015-02-07 09:36 .
drwxr-xr-x 4 root root 620 2015-02-07 09:36 ..
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 boot -> /dev/block/mmcblk0p7
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 cache -> /dev/block/mmcblk0p21
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 expdb -> /dev/block/mmcblk0p10
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 flashinfo -> /dev/block/mmcblk0p23
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 frp -> /dev/block/mmcblk0p17
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 keystore -> /dev/block/mmcblk0p14
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 lk -> /dev/block/mmcblk0p5
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 logo -> /dev/block/mmcblk0p9
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 metadata -> /dev/block/mmcblk0p19
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 nvdata -> /dev/block/mmcblk0p18
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 nvram -> /dev/block/mmcblk0p2
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 oemkeystore -> /dev/block/mmcblk0p12
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 para -> /dev/block/mmcblk0p6
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 proinfo -> /dev/block/mmcblk0p1
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 protect1 -> /dev/block/mmcblk0p3
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 protect2 -> /dev/block/mmcblk0p4
lrwxrwxrwx 1 root root  20 2015-02-07 09:36 recovery -> /dev/block/mmcblk0p8
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 seccfg -> /dev/block/mmcblk0p11
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 secro -> /dev/block/mmcblk0p13
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 system -> /dev/block/mmcblk0p20
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 tee1 -> /dev/block/mmcblk0p15
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 tee2 -> /dev/block/mmcblk0p16
lrwxrwxrwx 1 root root  21 2015-02-07 09:36 userdata -> /dev/block/mmcblk0p22
```

Dumped the first 0x4400 bytes of mmcblk0 (raw eMMC) in BROM mode.  
Parsed with gdisk (on my Host machine) → got the GPT.  
Built scatter file from start addresses & sizes.  

Example GPT dump (abridged):
```
╰─$ `sudo gdisk -l gpt.bin`
GPT fdisk (gdisk) version 1.0.10
Warning! Disk size is smaller than the main header indicates! Loading
secondary header from the last sector of the disk! You should use 'v' to
verify disk integrity, and perhaps options on the experts' menu to repair
the disk.
Caution: invalid backup GPT header, but valid main header; regenerating
backup header from main header.

Warning! Error 25 reading partition table for CRC check!
Warning! One or more CRCs don't match. You should repair the disk!
Main header: OK
Backup header: ERROR
Main partition table: OK
Backup partition table: ERROR

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: damaged

****************************************************************************
Caution: Found protective or hybrid MBR and corrupt GPT. Using GPT, but disk
verification and recovery are STRONGLY recommended.
****************************************************************************
Disk gpt.bin: 34 sectors, 17.0 KiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 00000000-0000-0000-0000-000000000000
Partition table holds up to 23 entries
Main partition table begins at sector 2 and ends at sector 7
First usable sector is 1024, last usable sector is 30776319
Partitions will be aligned on 1024-sector boundaries
Total free space is 0 sectors (0 bytes)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            1024            7167   3.0 MiB     0700  proinfo
   2            7168           17407   5.0 MiB     0700  nvram
   3           17408           37887   10.0 MiB    0700  protect1
   4           37888           58367   10.0 MiB    0700  protect2
   5           58368           62463   2.0 MiB     0700  lk
   6           62464           63487   512.0 KiB   0700  para
   7           63488           96255   16.0 MiB    0700  boot
   8           96256          129023   16.0 MiB    0700  recovery
   9          129024          145407   8.0 MiB     0700  logo
  10          145408          165887   10.0 MiB    0700  expdb
  11          165888          166911   512.0 KiB   0700  seccfg
  12          166912          171007   2.0 MiB     0700  oemkeystore
  13          171008          183295   6.0 MiB     0700  secro
  14          183296          199679   8.0 MiB     0700  keystore
  15          199680          209919   5.0 MiB     0700  tee1
  16          209920          220159   5.0 MiB     0700  tee2
  17          220160          222207   1024.0 KiB  0700  frp
  18          222208          287743   32.0 MiB    0700  nvdata
  19          287744          360447   35.5 MiB    0700  metadata
  20          360448         8552447   3.9 GiB     0700  system
  21         8552448         9371647   400.0 MiB   0700  cache
  22         9371648        30743551   10.2 GiB    0700  userdata
  23        30743552        30776319   16.0 MiB    0700  flashinfo
```

🎭 Conclusion

The Temu T7 is less “budget Android 14 tablet” and more 2015 zombie tablet in a trench coat:  
Specs spoofed harder than a bad Tinder profile.  
RAM inflated like a car dealership balloon man.  
SELinux wide open (thank you very much).  
But hey — with ADB, SP Flash Tool, and a DIY scatter file, it’s actually a fun playground for reverse engineering. Just don’t expect it to replace your Pixel anytime soon.  

⚡ TL;DR  
It’s Android 7 pretending to be Android 14.  
Real RAM: ~2 GB (not 16).  
SoC: MT6735P (a.k.a. a fossil).  
Spoofed props everywhere.  
Scatter file? Hand-rolled and ready for action.  
This repo = exposing cheap Android shenanigans, one fake spec at a time.  

I added my scatter_file for this:  

Android Catfish Edition 🐟 → looks like Android 14, but really Android 7 in disguise.  
Snapdragon Impostor 865 🎭 → “sus” as hell.  
Wish-droid™ 🛒 → because it feels straight outta Wish/Temu.  
RAMboozled 16GB 🤯 → promises 16, delivers 2.  
Project Masquerade 🎭 → because literally every prop is spoofed.  

Have fun, will add more stuff later. Peace y'all. :) <3

P.S. If you wanna see some funny goodness packed into the tablet. Go to the Dialer and type in `*#*#10000*#*#` in your dialer app. Have fun cosplaying your T7 Snapdragon 865 tablet with 1TB internal storage.
Please make sure you backup your preloader because I did not do that, nor dump it and list it within the scatter_file. If you have some time, make a pull request and I'll happily include it here. 
