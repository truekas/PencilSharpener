# Pencil Sharpener Explanation 
By disabling write protection on the flash chip through bridging pins 3 and 8, we can enter into developer mode. This also lets us boot into Sh1mmer.

However, at this point Sh1mmer cannot boot as the recovery kernel data key will fail to validate the shim and refuse to work. Fortunately, the correct keys are stored in a recovery file, so we can still extract and re-flash them.

Once we access the Sh1mmer bash console, we can modify the GBB flags. By adding the flag `0x80b3` FWMP will be ignored by the device. FWMP controls many important parts of the system, including enrollment data, what developer images work, and if users can boot from a USB.

Because of the matching kernel version, we can use a recovery image to downgrade the system from v130 to v124. The system may keyroll after recovery, but it can be bypassed again by re-flashing the correct keys.

By downgrading, we won't get switched back to verified mode through group policy, and we can access the VT2 console on the sign-in screen, where we can take ownership of the TPM module and clear all GBB flags. Since we have cleared all flags, we'll need to re-enable USB booting using `crossystem`.

Finally, we can start to unlock the read-only firmware by opening CCD. Due to a bug, this will switch the device back to verified mode. Because of this, we will have to boot into the recovery menu again to re-enable developer mode, and complete the process. The device will then be fully un-enrolled and under the user's control.
