# Pencil Sharpener Explanation

By disabling write protection on the flash chip through bridging pins 3 and 8, its possible to enter into developer mode. We can also now boot into Sh1mmer.

On keyrolled Ti50 systems, the changed keys will prevent Sh1mmer from booting. However, the correct keys are still stored in a recovery file, so they can be re-flashed to the system.

By using the Sh1mmer bash console, GBB flags can be modified. Adding the flag 0x80b3 will make it so FWMP will be ignored by the device. 

A recovery image can now be used to downgrade the system from v130 to v124. 

By downgrading, we won't get switched back to verified mode through group policy and by enabling flashrom, we wont get forced to re-enroll on the setup screen. Now, the VT2 console can be used, where we can take ownership of the TPM module and clear all GBB flags. 

Then, we can open CCD and start a read-only firmware unlock. Due to a bug, this will switch the device back to verified mode and we will have to boot into the recovery menu again to re-enable developer mode, and complete the unlock. 

The device should then be fully unenrolled. 
