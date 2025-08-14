# nvidia390-slackbuild

Slackbuilds for kernel modules, X driver, and nvidia utilities

This is a slightly changed version from slackbuilds.org, so credits go to Heinz Wiesinger, Edward W. Koenig, and Lenard Spencer, who are the authors of the original scripts.

# Changes:

- Adapted to build for XLibre, although should also work for X.Org, if the default `--x-module-path` is changed back from `/usr/lib??/xorg/modules/xlibre-25.0` to `/usr/lib??/xorg/modules/`

- Patches from AUR added to compile and work with recent kernels

- Logs are written to the respective directories and packages built are put there

# Prerequisites:

- You should either have XLibre with `IgnoreABI` and `Module` support in `OutputClass` and `TimerForce()` function reexported (get it at https://github.com/ONykyf/X11Libre-SlackBuild until these changes are merged into a stable version), or (in particular, for X.Org) put `IgnoreABI` option in `ServerFlags` (read `10-nvidia.conf` for details)

- `nouveau` kernel driver should be blacklisted (add `/etc/modprobe.d/BLACKLIST-nouveau.conf` to your system)

- `nvidia.ko`, `nvidia-drm.ko`, `nvidia-uvm.ko`, and `nvidia-modeset.ko` should be in initrd to be inserted at early boot

- parameters `nvidia-drm.modeset=1` and `nouveau.blackist=1` were passed to kernel through GRUB (although I am only sure about the first one that it is necessary and works)

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

After X session is started, the nvidia390 driver works nice, much better that nouveau (no lags, picture and fonts are clearer). Note that if You have an integrated card and want it to work in parallel at a separate seat (as i have `intel`),
then the second X instance for it should be run with `-modulepath /usr/lib64/xorg/modules` command-line option to prevent loading an nVidia GLX library instead of the XOrg stock version. This way You obtain different GLX
realizations working at different seats.


# nvidia470

Slackbuilds for building a newer 470 driver with kernel modules from the same authors are also added (slightly modified tom play well with XLibre).
I can only confirm that packages are built, but have no hardware supported by them, so cannot verify that they work (although I expect no problems).

Please try this, remarks and bug complaints (if any) are welcome!