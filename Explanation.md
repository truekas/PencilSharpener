# Pencil Sharpener Explanation 

By bridging the pins on the flash chip, write protection is temporarily disabled. This lets us enter developer mode. We can now try to boot into Sh1mmer. But because Google has changed the keys for Ti50 systems, we will get a `no valid image error`.

Thankfully, the original unchanged shim keys are stored in a recovery file. We can then extract and flash them to the system, rolling back Google's changes and booting into Sh1mmer. 

Now that we have access to a bash console, we can start modifying some GBB flags (in this case, we use `0x80b3` to ignore FWMP). FWMP controls many things, including what dev images can run, enrollment data, and whether USB boot works.

We can now use a recovery image to roll back the system from v130 to v124 because of the matching kernel version. We can then access the VT2 console on the sign-in screen to take ownership of the TPM module and remove all GBB flags on the flash chip (which holds enrollment data). Because the existing flags were cleared, including the one we set to allow USB booting, we do need to re-enable it again using `crosssystem`. 

Now, we can start [unlocking](https://www.chromium.org/chromium-os/developer-library/guides/device/ro-firmware-unlock/) the read-only firmware. We will have to open CCD, and the device will forget it is in developer mode due to a bug. We will have to boot back into the recovery menu and re-enable dev mode to access the VT2 console again. Now, it is possible to permanently modify the read-only firmware, and the device should be fully unenrolled.



