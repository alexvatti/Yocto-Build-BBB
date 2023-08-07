# Yocto-Build-BBB
Yocto Build Steps for Beagle Bone Black

# Build Steps
git clone -b dunfell git://git.yoctoproject.org/poky.git poky-dunfell


cd poky-dunfell

git clone -b dunfell git://git.openembedded.org/meta-openembedded

git clone -b dunfell https://github.com/meta-qt5/meta-qt5.git

git clone -b dunfell git://git.yoctoproject.org/meta-security.git

git clone -b dunfell https://github.com/jumpnow/meta-jumpnow.git

git clone -b dunfell https://github.com/jumpnow/meta-bbb.git

# Change 1:
poky-dunfell$ git diff

diff --git a/.templateconf b/.templateconf

index 0fe6f82503..e381111443 100644

--- a/.templateconf

+++ b/.templateconf

@@ -1,2 +1,2 @@

# Template settings

-TEMPLATECONF=${TEMPLATECONF:-meta-poky/conf}

+TEMPLATECONF=${TEMPLATECONF:-meta-bbb/conf}


# Change 2:
$ cd meta-bbb
$ git diff
diff --git a/conf/bblayers.conf.sample b/conf/bblayers.conf.sampleindex 16d1364..a9f1652 100644
--- a/conf/bblayers.conf.sample
+++ b/conf/bblayers.conf.sample
@@ -6,14 +6,14 @@ BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
- ${HOME}/poky-dunfell/meta \
- ${HOME}/poky-dunfell/meta-poky \
- ${HOME}/poky-dunfell/meta-openembedded/meta-oe \
- ${HOME}/poky-dunfell/meta-openembedded/meta-networking \
- ${HOME}/poky-dunfell/meta-openembedded/meta-perl \
- ${HOME}/poky-dunfell/meta-openembedded/meta-python \
- ${HOME}/poky-dunfell/meta-qt5 \
- ${HOME}/poky-dunfell/meta-security \
- ${HOME}/poky-dunfell/meta-jumpnow \
- ${HOME}/bbb/meta-bbb \
+ ##OEROOT##/meta \
+ ##OEROOT##/meta-poky \
+ ##OEROOT##/meta-openembedded/meta-oe \
+ ##OEROOT##/meta-openembedded/meta-networking \
+ ##OEROOT##/meta-openembedded/meta-perl \
+ ##OEROOT##/meta-openembedded/meta-python \
+ ##OEROOT##/meta-qt5 \
+ ##OEROOT##/meta-security \
+ ##OEROOT##/meta-jumpnow \
+ ##OEROOT##/meta-bbb \
"
# Build:

cd ../poky-dunfell
$ source oe-init-build-env ../build
build$ bitbake console-image

# Images
Copy the following - from the following directory
tmp/deploy/images/beaglebone/
1. console-image-beaglebone-20220608054739.rootfs.tar.xz
2. MLO-beaglebone-2020.01-r0
3. u-boot-beaglebone.img

# Prepare-SD Card
mk2parts.sh (We can find this scripts)
meta-bbb/scripts$ sudo ./mk2parts.sh sdb
lsblk is convenient for finding the microSD card.

# Copying Images
You will need to create a mount point on your workstation for the copy scripts to use.
~$ sudo mkdir /media/card
sudo mkfs.vfat /dev/sda1
sudo mount /dev/sda1 /media/card
sudo cp u-boot-beaglebone.img /media/card/u-boot.img
sudo cp uEnv.txt /media/card
sudo cp MLO-beaglebone-2020.01-r0 /media/card/MLO
sudo umount /dev/sda1

sudo mkfs.ext4 -q -L ROOT /dev/sda2
sudo mount /dev/sda2 /media/card
sudo tar -C /media/card -xJf console-image-beaglebone-20220609043454.rootfs.tar.xz
mkdir -p /media/card/var/lib/urandom
sudo dd if=/dev/urandom of=/media/card/var/lib/urandom/random-seed bs=512 count=1
sudo chmod 600 /media/card/var/lib/urandom/random-seed
sudo -E bash -c 'echo "beaglebone" > /media/card/etc/hostname'
sudo cp ./interfaces /media/card/etc/network/interfaces
sudo cp ./wpa_supplicant.conf /media/card/etc/wpa_supplicant.conf
sudo umount /dev/sda2

# Booting from SD
Booting from the SD card
The default behavior of the beaglebone is to boot from the eMMC first if it finds a bootloader there.
Holding the S2 switch down when the bootloader starts will cause the BBB to try booting from the
SD card first. The S2 switch is above the SD card holder.
User name : root
Password: jumpnowtek

#BOOT LOG / REFERENCE
U-Boot SPL 2020.01 (Jan 06 2020 - 20:56:31 +0000)
Trying to boot from MMC1


U-Boot 2020.01 (Jan 06 2020 - 20:56:31 +0000)

CPU  : AM335X-GP rev 2.1
Model: TI AM335x BeagleBone Black
DRAM:  512 MiB
WDT:   Started with servicing (60s timeout)
NAND:  0 MiB
MMC:   OMAP SD/MMC: 0, OMAP SD/MMC: 1
Loading Environment from FAT... *** Warning - bad CRC, using default environment

<ethaddr> not set. Validating first E-fuse MAC
Net:   eth0: ethernet@4a100000
Warning: usb_ether MAC addresses don't match:
Address in ROM is          de:ad:be:ef:00:01
Address in environment is  38:d2:69:4e:1c:a3
, eth1: usb_ether
Hit any key to stop autoboot:  0 
switch to partitions #0, OK
mmc0 is current device
SD/MMC found on device 0
1398 bytes read in 2 ms (682.6 KiB/s)
Loaded env from uEnv.txt
Importing environment from mmc0 ...
Running uenvcmd ...
** Invalid partition 5 **
** Invalid partition 5 **
Using root partition 0:2
61895 bytes read in 7 ms (8.4 MiB/s)
Loaded am335x-boneblack.dtb
4924568 bytes read in 317 ms (14.8 MiB/s)
## Flattened Device Tree blob at 88000000
   Booting using the fdt blob at 0x88000000
   Loading Device Tree to 8ffed000, end 8ffff1c6 ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 5.9.16-jumpnow (oe-user@oe-host) (arm-poky-linux-gnueabi-gcc (GCC) 9.5.0, GNU ld (GNU Binutils) 2.34.0.20200910)0
[    0.000000] CPU: ARMv7 Processor [413fc082] revision 2 (ARMv7), cr=10c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: TI AM335x BeagleBone Black
[    0.000000] Memory policy: Data cache writeback
[    0.000000] cma: Reserved 16 MiB at 0x9e800000
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000080000000-0x000000009fefffff]
[    0.000000]   HighMem  empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080000000-0x000000009fefffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080000000-0x000000009fefffff]
[    0.000000] CPU: All CPU(s) started in SVC mode.
[    0.000000] AM335X ES2.1 (sgx neon)
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 129794
[    0.000000] **Kernel command line: console=ttyO0,115200n8 root=/dev/mmcblk1p2 rw rootfstype=ext4 rootwait consoleblank=0**
[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:on, heap free:off
[    0.000000] Memory: 483660K/523264K available (7168K kernel code, 528K rwdata, 2476K rodata, 1024K init, 6584K bss, 23220K reserved, 16384)
[    0.000000] random: get_random_u32 called from __kmem_cache_create+0x20/0x35c with crng_init=0
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] Running RCU self tests
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] IRQ: Found an INTC at 0x(ptrval) (revision 5.0) with 128 interrupts
[    0.000000] TI gptimer clocksource: always-on /ocp/interconnect@44c00000/segment@200000/target-module@31000
[    0.000010] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
[    0.000035] clocksource: dmtimer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000704] TI gptimer clockevent: 24000000 Hz at /ocp/interconnect@48000000/segment@0/target-module@40000
[    0.002100] Console: colour dummy device 80x30
[    0.002164] Lock dependency validator: Copyright (c) 2006 Red Hat, Inc., Ingo Molnar
[    0.002181] ... MAX_LOCKDEP_SUBCLASSES:  8
[    0.002197] ... MAX_LOCK_DEPTH:          48
[    0.002212] ... MAX_LOCKDEP_KEYS:        8192
[    0.002227] ... CLASSHASH_SIZE:          4096
[    0.002243] ... MAX_LOCKDEP_ENTRIES:     32768
[    0.002258] ... MAX_LOCKDEP_CHAINS:      65536
[    0.002273] ... CHAINHASH_SIZE:          32768
[    0.002288]  memory used by lock dependency info: 4061 kB
[    0.002304]  memory used for stack traces: 2112 kB
[    0.002320]  per task-struct memory footprint: 1536 bytes
[    0.002483] Calibrating delay loop... 996.14 BogoMIPS (lpj=4980736)
[    0.080839] pid_max: default: 32768 minimum: 301
[    0.081407] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.081435] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.084476] CPU: Testing write buffer coherency: ok
[    0.084702] CPU0: Spectre v2: using BPIALL workaround
[    0.086848] Setting up static identity map for 0x80100000 - 0x80100060
[    0.089238] devtmpfs: initialized
[    0.140745] VFP support v0.3: implementor 41 architecture 3 part 30 variant c rev 3
[    0.141758] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.141853] futex hash table entries: 256 (order: 1, 12288 bytes, linear)
[    0.143293] pinctrl core: initialized pinctrl subsystem
[    0.148347] NET: Registered protocol family 16
[    0.156351] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.200911] l3-aon-clkctrl:0000:0: failed to disable
[    0.206089] thermal_sys: Registered thermal governor 'fair_share'
[    0.206104] thermal_sys: Registered thermal governor 'step_wise'
[    0.206144] thermal_sys: Registered thermal governor 'user_space'
[    0.206607] cpuidle: using governor ladder
[    0.206700] cpuidle: using governor menu
[    2.571082] random: fast init done
[    2.952454] No ATAGs?
[    2.952497] hw-breakpoint: debug architecture 0x4 unsupported.
[    3.156031] usbcore: registered new interface driver usbfs
[    3.156311] usbcore: registered new interface driver hub
[    3.156527] usbcore: registered new device driver usb
[    3.186061] clocksource: Switched to clocksource dmtimer
[    3.364989] VFS: Disk quotas dquot_6.6.0
[    3.365125] VFS: Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
[    3.455221] NET: Registered protocol family 2
[    3.457577] tcp_listen_portaddr_hash hash table entries: 256 (order: 1, 11264 bytes, linear)
[    3.457710] TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    3.457977] TCP bind hash table entries: 4096 (order: 5, 163840 bytes, linear)
[    3.458560] TCP: Hash tables configured (established 4096 bind 4096)
[    3.458977] UDP hash table entries: 256 (order: 2, 24576 bytes, linear)
[    3.459119] UDP-Lite hash table entries: 256 (order: 2, 24576 bytes, linear)
[    3.459647] NET: Registered protocol family 1
[    3.461923] hw perfevents: enabled with armv7_cortex_a8 PMU driver, 5 counters available
[    3.476592] Initialise system trusted keyrings
[    3.483899] workingset: timestamp_bits=30 max_order=17 bucket_order=0
[    3.623513] Key type asymmetric registered
[    3.623696] Asymmetric key parser 'x509' registered
[    3.623874] io scheduler mq-deadline registered
[    3.623904] io scheduler kyber registered
[    4.010268] gpiochip_add_data_with_key: GPIOs 0..31 (gpio-0-31) failed to register, -517
[    4.010343] omap_gpio 44e07000.gpio: Could not register gpio chip -517
[    4.616619] gpiochip_add_data_with_key: GPIOs 0..31 (gpio-0-31) failed to register, -517
[    4.616673] omap_gpio 4804c000.gpio: Could not register gpio chip -517
[    4.740727] gpiochip_add_data_with_key: GPIOs 0..31 (gpio-0-31) failed to register, -517
[    4.740783] omap_gpio 481ac000.gpio: Could not register gpio chip -517
[    4.770157] gpiochip_add_data_with_key: GPIOs 0..31 (gpio-0-31) failed to register, -517
[    4.770208] omap_gpio 481ae000.gpio: Could not register gpio chip -517
[    4.941740] debugfs: Directory '49000000.dma' with parent 'dmaengine' already present!
[    4.941931] edma 49000000.dma: TI EDMA DMA engine driver
[    5.143360] pinctrl-single 44e10800.pinmux: 142 pins, size 568
[    5.165628] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    5.178962] omap_uart 44e09000.serial: no wakeirq for uart0
[    5.180692] 44e09000.serial: ttyO0 at MMIO 0x44e09000 (irq = 20, base_baud = 3000000) is a OMAP UART0
[    5.836122] printk: console [ttyO0] enabled
[    5.850780] omap_rng 48310000.rng: Random Number Generator ver. 20
[    5.859722] random: crng init done
[    5.865433] loop: module loaded
[    5.872283] mtdoops: mtd device (mtddev=name/number) must be supplied
[    5.883399] libphy: Fixed MDIO Bus: probed
[    5.888414] CAN device driver interface
[    5.966148] davinci_mdio 4a101000.mdio: davinci mdio revision 1.6, bus freq 1000000
[    5.974345] libphy: 4a101000.mdio: probed
[    5.987290] davinci_mdio 4a101000.mdio: phy[0]: device 4a101000.mdio:00, driver SMSC LAN8710/LAN8720
[    5.998381] cpsw 4a100000.ethernet: initialized cpsw ale version 1.4
[    6.005058] cpsw 4a100000.ethernet: ALE Table size 1024
[    6.014221] cpsw 4a100000.ethernet: Detected MACID = 38:d2:69:4e:1c:a1
[    6.025788] usbcore: registered new interface driver asix
[    6.031771] usbcore: registered new interface driver ax88179_178a
[    6.038331] usbcore: registered new interface driver cdc_ether
[    6.044603] usbcore: registered new interface driver smsc95xx
[    6.050779] usbcore: registered new interface driver net1080
[    6.056922] usbcore: registered new interface driver cdc_subset
[    6.063237] usbcore: registered new interface driver zaurus
[    6.069304] usbcore: registered new interface driver cdc_ncm
[    6.079040] am335x-phy-driver 47401300.usb-phy: supply vcc not found, using dummy regulator
[    6.108249] am335x-phy-driver 47401b00.usb-phy: supply vcc not found, using dummy regulator
[    6.122810] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    6.129785] ehci-omap: OMAP-EHCI Host Controller driver
[    6.136324] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    6.143057] usbcore: registered new interface driver cdc_wdm
[    6.149241] usbcore: registered new interface driver usbtest
[    6.164617] musb-hdrc musb-hdrc.1: MUSB HDRC host driver
[    6.171149] musb-hdrc musb-hdrc.1: new USB bus registered, assigned bus number 1
[    6.182951] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 5.09
[    6.191847] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    6.199515] usb usb1: Product: MUSB HDRC host driver
[    6.204725] usb usb1: Manufacturer: Linux 5.9.16-jumpnow musb-hcd
[    6.211158] usb usb1: SerialNumber: musb-hdrc.1
[    6.221208] hub 1-0:1.0: USB hub found
[    6.227151] hub 1-0:1.0: 1 port detected
[    6.244783] mousedev: PS/2 mouse device common for all mice
[    6.264680] omap_rtc 44e3e000.rtc: registered as rtc0
[    6.270192] omap_rtc 44e3e000.rtc: setting system clock to 2000-01-01T00:00:00 UTC (946684800)
[    6.280816] i2c /dev entries driver
[    6.295639] omap_wdt: OMAP Watchdog Timer Rev 0x01: initial timeout 60 sec
[    6.305784] cpu cpu0: OPP-v2 not supported, cpufreq-dt will attempt to use legacy tables.
[    6.317022] sdhci: Secure Digital Host Controller Interface driver
[    6.323514] sdhci: Copyright(c) Pierre Ossman
[    6.328146] sdhci-pltfm: SDHCI platform and OF driver helper
[    6.350506] sdhci-omap 481d8000.mmc: supply vqmmc not found, using dummy regulator
[    6.396722] mmc0: SDHCI controller on 481d8000.mmc [481d8000.mmc] using External DMA
[    6.408022] ledtrig-cpu: registered to indicate activity on CPUs
[    6.418970] usbcore: registered new interface driver usbhid
[    6.424829] usbhid: USB HID core driver
[    6.431169] Initializing XFRM netlink socket
[    6.435831] NET: Registered protocol family 17
[    6.440646] can: controller area network core (rev 20170425 abi 9)
[    6.447811] NET: Registered protocol family 29
[    6.452492] can: raw protocol (rev 20170425)
[    6.457171] can: broadcast manager protocol (rev 20170425 t)
[    6.463202] Key type dns_resolver registered
[    6.469034] oprofile: using arm/armv7
[    6.472999] ThumbEE CPU extension supported.
[    6.477568] omap_voltage_late_init: Voltage driver support not added
[    6.502140] Loading compiled-in X.509 certificates
[    6.587668] OMAP GPIO hardware version 0.1
[    6.638604] tps65217-pmic: Failed to locate of_node [id: -1]
[    6.652880] mmc0: new high speed MMC card at address 0001
[    6.669596] mmcblk0: mmc0:0001 M62704 3.56 GiB 
[    6.675262] mmcblk0boot0: mmc0:0001 M62704 partition 1 2.00 MiB
[    6.682444] mmcblk0boot1: mmc0:0001 M62704 partition 2 2.00 MiB
[    6.694073] mmcblk0rpmb: mmc0:0001 M62704 partition 3 512 KiB, chardev (250:0)
[    6.715814] tps65217-bl: Failed to locate of_node [id: -1]
[    6.740134]  mmcblk0: p1 p2 p3 p4 < p5 p6 >
[    6.748825] tps65217 0-0024: TPS65217 ID 0xe version 1.2
[    6.755799] at24 0-0050: supply vcc not found, using dummy regulator
[    6.794753] at24 0-0050: 32768 byte 24c256 EEPROM, writable, 1 bytes/write
[    6.804297] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
[    6.849242] omap_gpio 44e07000.gpio: Could not set line 6 debounce to 200000 microseconds (-22)
[    6.858499] sdhci-omap 48060000.mmc: Got CD GPIO
[    6.864032] sdhci-omap 48060000.mmc: supply vqmmc not found, using dummy regulator
[    6.915313] mmc1: SDHCI controller on 48060000.mmc [48060000.mmc] using External DMA
[    6.945774] Waiting for root device /dev/mmcblk1p2...
[    6.979317] mmc1: new high speed SDHC card at address 21d1
[    6.988106] mmcblk1: mmc1:21d1 APPSD 3.75 GiB 
[    6.997813]  mmcblk1: p1 p2
[    7.033527] EXT4-fs (mmcblk1p2): mounted filesystem with ordered data mode. Opts: (null)
[    7.042515] VFS: Mounted root (ext4 filesystem) on device 179:26.
[    7.051320] devtmpfs: mounted
[    7.056586] Freeing unused kernel memory: 1024K
[    7.062114] Run /sbin/init as init process
INIT: version 2.96 booting
Starting udev
[    8.030488] udevd[1016]: starting version 3.2.9
[    8.132303] udevd[1017]: starting eudev-3.2.9
[    9.273008] tda998x 0-0070: found TDA19988
[    9.644644] tilcdc 4830e000.lcdc: bound 0-0070 (ops tda998x_ops [tda998x])
[    9.808411] [drm] Initialized tilcdc 1.0.0 20121205 for 4830e000.lcdc on minor 0
[    9.924556] tilcdc 4830e000.lcdc: [drm] Cannot find any crtc or sizes
[    9.947015] tilcdc 4830e000.lcdc: [drm] Cannot find any crtc or sizes
[   10.086666] omap-sham 53100000.sham: hw accel on OMAP rev 4.3
[   10.140664] omap-aes 53500000.aes: OMAP AES hw accel rev: 3.2
[   10.191650] omap-aes 53500000.aes: will run requests pump with realtime priority
[   10.739921] EXT4-fs (mmcblk1p2): re-mounted. Opts: 
Sat Jan  1 00:05:52 UTC 2000                                                        
Starting firewall                                          
[   13.712446] NET: Registered protocol family 10                                   
[   13.722619] Segment Routing with IPv6               
INIT: Entering runlevel: 5
Configuring network interfaces... udhcpc: started, v1.31.1
[   14.397553] cpsw 4a100000.ethernet: initializing cpsw version 1.12 (0)
[   14.498208] SMSC LAN8710/LAN8720 4a101000.mdio:00: attached PHY driver [SMSC LAN8710/LAN8720] (mii_bus:phy_addr=4a101000.mdio:00, irq=POLL)
udhcpc: sending discover
udhcpc: sending discover
udhcpc: sending discover
udhcpc: no lease, failing
ifup: failed to bring up eth0
Starting system message bus: dbus.
Starting OpenBSD Secure Shell server: sshd
done.
Starting ntpd: done
Starting syslogd/klogd: done
There is no /dev/mmcblk1p5 partition

Poky (Yocto Project Reference Distro) 3.1.27 beaglebone ttyO0

beaglebone login: root
Password: 
root@beaglebone:~# 
root@beaglebone:~# 
root@beaglebone:~# 
