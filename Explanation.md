# Pencil Sharpener Explanation 

To disable write protection on the flash chip and enter developer mode, we need to bridge the pins on the chip. While we can attempt to boot into Sh1mmer, we won’t be able to as Google has rolled keys for Ti50 systems.

Fortunately, the correct keys are stored in a recovery file, allowing us to extract and flash them to the system. This will enable us to use Sh1mmer.

Once we access the Sh1mmer bash console, we can modify the GBB flags. We will add the flag `0x80b3` so FWMP will be ignored. FWMP controls many important parts of the system, including enrollment data, developer images, and if users can USB boot.

Because of the matching kernel version, we can use a recovery image to downgrade the system from v130 to v124. This lets us access the VT2 console on the sign-in screen, where we can take ownership of the TPM module and clear all GBB flags on the flash chip, which holds enrollment data. Since we have cleared all flags, we'll need to re-enable USB booting using `crosssystem`.

Finally, we can unlock the read-only firmware by opening CCD. This will switch the device back to verified mode. Because of this, we will have to boot into the recovery menu again to re-enable developer mode, which allows us to modify the read-only firmware. The device will then be fully un-enrolled and under the user's control.
