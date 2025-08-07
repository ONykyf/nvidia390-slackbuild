# nvidia390-slackbuild

Slackbuilds for kernel modules, X driver, and nvidia utilities

This is a slightly changed version from slackbuilds.org, so credits go to Heinz Wiesinger and Edward W. Koenig, who are the authors of the original scripts.

# Changes:

- Adapted to build for XLibre, although should also work for X.Org, if the default `--x-module-path` is changed back from `/usr/lib??/xorg/modules/xlibre-25.0` back to `/usr/lib??/xorg/modules/`

- Patches from AUR added to compile and work with recent kernels

- Logs are written to the respective directories and packages built are put there

# Prerequisites:

- `TimerForce()`, which was unexported in XLibre, should be reexported (apply the patch to xlibre-server)

- `nouveau` kernel driver should be blacklisted (add `/etc/modprobe.d/BLACKLIST-nouveau.conf` to your system)

- `nvidia.ko`, `nvidia-drm.ko`, `nvidia-uvm.ko`, and `nvidia-modeset.ko` should be in initrd to be inserted at early boot

- You should either have XLibre with `IgnoreABI` support in `OutputClass`, or (in particular, for X.Org) put this option in `ServerFlags`

- parameters `nvidia-drm.modeset=1`, `fbdev=1`, and `nouveau.blackist=1` were passed to kernel through GRUB (although I am only sure about the first one that it is necessary and works)

# Status and caveats

There is a warning about "tainted kernel" in `dmesg` output.

After a framebuffer for the second (integrated) intel card starts to initialize, the text console gets black for a long (ca 30 seconds) and returns only before X is started.

Maybe the pause is long because something goes wrong with networking. When Xserver is started, `nvidia-drm` complains about failure to grab drm device ownership, which is a long-term issue for NVidia:
```
[   22.205067] RPL Segment Routing with IPv6
[   22.205090] In-situ OAM (IOAM) with IPv6
[   35.776277] RTL8211E Gigabit Ethernet r8169-0-300:00: attached PHY driver (mii_bus:phy_addr=r8169-0-300:00, irq=MAC)
[   35.982801] r8169 0000:03:00.0 eth0: Link is Down
[   38.094386] r8169 0000:03:00.0 eth0: Link is Up - 1Gbps/Full - flow control rx/tx
[   38.526457] 8021q: 802.1Q VLAN Support v1.8
[   38.906403] cfg80211: Loading compiled-in X.509 certificates for regulatory database
[   38.906624] Loaded X.509 cert sforshee: 00b28ddf47aef9cea7
[   38.906804] Loaded X.509 cert wens: 61c038651aabdcf94bd0ac7ff06c7248db18c600
[   39.677561] NET: Registered PF_QIPCRTR protocol family
[   70.657424] [drm:drm_new_set_master [drm]] *ERROR* [nvidia-drm] [GPU ID 0x00000100] Failed to grab modeset ownership
[   82.852261] CIFS: Attempting to mount //nigga/family-homes-write
[   83.284368] CIFS: SMB3.11 POSIX Extensions are experimental
```

After X session is started, the nvidia390 driver works nice, much better that nouveau (no lags, picture and fonts are clearer), although `libglx.so.390.157` from NVidia fails to load, and X screen is black:
```
[2025-08-07 10:03:30] (==) NVIDIA(0): Depth 24, (==) framebuffer bpp 32
[2025-08-07 10:03:30] (==) NVIDIA(0): RGB weight 888
[2025-08-07 10:03:30] (==) NVIDIA(0): Default visual is TrueColor
[2025-08-07 10:03:30] (==) NVIDIA(0): Using gamma correction (1.00, 1.00, 1.00)
[2025-08-07 10:03:30] (II) Applying OutputClass "NVidia" options to /dev/dri/card0
[2025-08-07 10:03:30] (**) NVIDIA(0): Enabling 2D acceleration
[2025-08-07 10:03:30] (EE) NVIDIA(0): Failed to initialize the GLX module; please check in your X
[2025-08-07 10:03:30] (EE) NVIDIA(0):     log file that the GLX module has been loaded in your X
[2025-08-07 10:03:30] (EE) NVIDIA(0):     server, and that the module is the NVIDIA GLX module.  If
[2025-08-07 10:03:30] (EE) NVIDIA(0):     you continue to encounter problems, Please try
[2025-08-07 10:03:30] (EE) NVIDIA(0):     reinstalling the NVIDIA driver.
```
probably because of the presence of an integrated card. If the stock `libglx.so` from xserver is used, X starts, even glxgears work if the integrated intel card is initialised, but glxinfo on nvidia shows only intel.
```
[2025-08-07 10:03:29] (II) xfree86: Adding drm device (/dev/dri/card0)
[2025-08-07 10:03:29] (II) Platform probe for /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/drm/card0
[2025-08-07 10:03:29] (II) xfree86: Adding drm device (/dev/dri/card1)
[2025-08-07 10:03:29] (II) Platform probe for /sys/devices/pci0000:00/0000:00:02.0/drm/card1
[2025-08-07 10:03:29] (**) OutputClass "NVidia" sets nvidia to ignore ABI for "/dev/dri/card0" managed by nvidia-drm
[2025-08-07 10:03:29] (--) PCI:*(1@0:0:0) 10de:1201:1043:83b5 rev 161, Mem @ 0xf4000000/33554432, 0xe0000000/134217728, 0xe8000000/67108864, I/O @ 0xe000/128, BIOS @ 0x????????/131072
[2025-08-07 10:03:29] (II) Open ACPI successful (/var/run/acpid.socket)
[2025-08-07 10:03:29] (II) LoadModule: "glx"
[2025-08-07 10:03:29] (II) Loading /usr/lib64/xorg/modules/xlibre-25.0/extensions/libglx.so
[2025-08-07 10:03:29] (II) Module glx: vendor="X.Org Foundation"
```

This issue likely would not arise if two cards were not present and active.
