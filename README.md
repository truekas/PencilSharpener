# Pencil Sharpener
> [!CAUTION]  
> DURING THE EXPLOIT, MAKE SURE THE CHROMEBOOK STAYS PLUGGED INTO A CHARGER AT ALL TIMES (EXCEPT WHILE BRIDGING THE WP PINS). OTHERWISE THE SYSTEM MAY BRICK

> [!IMPORTANT]
> This exploit is for *non-factory keyrolled* Ti50 systems only, meaning that your Chromebook had its *shim keys changed in an update* and not during its assembly. 

## Hey There!
Google has released changes in version 136 (as of May 13, 2025) that will change how you can use Pencil Sharpener. Now, powerwashing your device even after it has been unenrolled will require you to redo the exploit. As a result, switching to other release channels (beta, canary) is impossible. This write-up exists only for educational purposes. Please do not damage or tamper with a system that does not belong to you.

## Introduction 
Pencil Sharpener is an exploit that allows users to unenroll *non-factory keyrolled* Ti50 Chromebooks through a modified version of the pencil method. This works because the Google Security Chip does not verify device hashes until the system completely loses power, allowing users to temporarily bypass validation checks and disable RO verification. 

You can watch our proof of concept video on Odysee if you need a visual demonstration of the exploit:
<br>
[![Video Demo](https://github.com/truekas/PencilSharpener/blob/main/src/Cover.png?raw=true)](https://ody.sh/xySDCFhvHi)
  
## The Exploit
**The materials you need:** (we reccommend [this kit](https://www.amazon.com/AiTrip-EEPROM-Programmer-CH341A-Adapter/dp/B07VNVVXW6) for SOIC-8 chips and [this one](https://a.co/d/5dswJ8O) for WSON-8)

- SOIC-8 or WSON-8 chip clip
- Paperclip or Safety Pin
- Linux system with flashrom installed
- A screwdriver
- Sh1mmer image for your board
- Working charger
- Ch341a Flash Programmer
- A ChromeOS recovery USB for v124

First, fully power off and unplug your device, flip it over, and open the back to access the mainboard.

> [!TIP] 
> **How to Setup Your Chip clip:**
> \
> \
> Take your chip clip, and a safety pin (recommended) or paperclip. If you are using a safety pin, cut off the bigger side.\
> Put the paperclip or safety pin into holes 3 and 8. To find these, find the red wire. This is pin 1. From there, you can find the other pins.
> With pin 1 in the top left, pin 3 would be the 3rd pin in the top row, and pin 8 would be directly under pin 1.

Then, disconnect the battery and locate your Flash Chip. Bridge pins 3 and 8.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/2.png?raw=true" alt="2.png"/>

Flip your laptop to its side with the charging port facing up and the pins still bridged. Plug in your device and push `esc + refresh + power` to enter the device recovery menu, then press `ctrl + d`. As soon as the screen goes black, press `esc + refresh + power` again. 

Insert your Sh1mmer USB and boot onto it. You may get a `no valid image` error. If this happens, you need to re-flash the correct keys to the device using instructions in the [rolled keys](#fixing-rolled-keys) section.

After booting into Sh1mmer, you should choose `utilities > unenroll`. It should give an error. Open the bash console WHILE MAKING SURE THE PINS ARE STILL BRIDGED and run:
```
flashrom --wp-disable
/usr/share/vboot/bin/set_gbb_flags.sh 0x80b3
flashrom --wp-enable
```
> [!IMPORTANT]
> On newer versions, if you get an error saying "owner has disabled downgrading" or "verified images only" you must disable rootfs verification in sh1mmer.

Hit `esc + refresh + power` and boot onto your v124 recovery USB. If you have any issues before or after the recovery process, follow the [rolled keys steps](#fixing-rolled-keys).

After the recovery is complete, boot into ChromeOS. Then, on the sign-in screen switch to VT2 by pressing `ctrl + alt + f2`. 

If you are prompted to login on the console, try to log in as `chronos` with no password, and elevate to root by using `sudo -i`. If that does not work, you can try logging in as `root` and then using `test0000` as your password. After you have access to the shell, run the following commands:
```
tpm_manager_client take_ownership
cryptohome --action=remove_firmware_management_parameters
crossystem dev_boot_usb=1
```

Reconnect the battery to the motherboard, and run `gsctool -a -o`. Follow the prompts to push the power button and the system should automatically reboot. When the device turns back on, re-open the recovery menu and re-enable devmode.

Afterward, go back to the VT2 console, run `gsctool -a -I AllowUnverifiedRo:always`, and the device should be unenrolled.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/unenrolled.png?raw=true" alt="unenrolled"/>

## Fixing Rolled Keys
**IMPORTANT: THIS WILL NOT UNROLL FACTORY ROLLED KEYS!!**
While using Pencil Sharpener, you may be unable to boot onto Sh1mmer or your recovery USB. This is because your system's shim keys have been changed in an update.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/rolledkeys.png?raw=true" alt="ch341a and shell"/>

This issue is fixed by re-flashing the correct keys to the system. Here's how to do it:

First, take your ch341a flash programmer and attach it to your chip clip (the red wire connects to number 1 on the ch341a). Take the end of your chip clip, and re-attach it to your flash chip. Now [connect](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html#prepping-to-flash) using your Linux system and run the following commands: 

```bash
flashrom --wp-disable
futility gbb -p ch341a_spi -r file.bin
futility gbb -p ch341a_spi -s -r file.bin
flashrom --wp-enable
```

## Re-Enrolling
You can re-enroll your device by accessing a VT2 shell, typing `vpd -i RW_VPD -s check_enrollment=1`, and then powerwashing the device using `CTRL + ALT + SHIFT + R`.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/enrolled.png?raw=true" alt="enrolled screen"/>

## Citations
[Breaking chromeOS's enrollment security model: A postmortem](https://blog.coolelectronics.me/breaking-cros-6/)
<br>
[Flashing GBB Flags](https://appleflyers-blog.vercel.app/blog/gbbflagflash)
<br>
[vboot_reference futility flags](https://chromium.googlesource.com/chromiumos/platform/vboot_reference/+/refs/heads/main/futility/docs/cmd_gbb_utility.md)
<br>
[Disabling Firmware Write Protection | MrChromebox.tech](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html)
<br>
[Unbricking/Flashing with a ch341a USB programmer | Chrultrabook Docs](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html)
<br>
[Verified Boot](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot/)
<br>
[Firmware Boot and Recovery](https://www.chromium.org/chromium-os/chromiumos-design-docs/firmware-boot-and-recovery/)
<br>
[Verified Boot Data Structures](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot-data-structures/)
<br>
[CrOS EC (Embedded Controller) - Google Security Chip (GSC) Case Closed Debugging (CCD)](https://chromium.googlesource.com/chromiumos/platform/ec/+/cr50_stab/docs/case_closed_debugging_gsc.md)
<br>
[hdctools: Chrome OS Hardware Debug & Control Tools - Closed Case Debug (CCD)](https://chromium.googlesource.com/chromiumos/third_party/hdctools/+/HEAD/docs/ccd.md)
<br>
[CrOS EC (Embedded Controller) - Google Security Chip (GSC) Case Closed Debugging (CCD)](https://chromium.googlesource.com/chromiumos/platform/ec/+/fe6ca90e/docs/case_closed_debugging_cr50.md)
<br>
[Read-only firmware unlock on 2023+ devices](https://www.chromium.org/chromium-os/developer-library/guides/device/ro-firmware-unlock/)
<br>
[Firmware Write Protection on ChromeOS Devices | MrChromebox.tech](https://docs.mrchromebox.tech/docs/firmware/wp/)
<br>
[Firmware Management Parameters](https://www.chromium.org/chromium-os/fwmp/)
<br>
[GBB flag-inator](https://binbashbanana.github.io/gbbflaginator/)
<br>
[CrOS EC (Embedded Controller) | Software Sync](https://chromium.googlesource.com/chromiumos/platform/ec/+/HEAD/README.md#Preventing-the-RW-EC-firmware-from-being-overwritten-by-Software-Sync-at-boot)
<br>
[Chromium OS Docs - Firmware Test Manual](https://chromium.googlesource.com/chromiumos/docs/+/master/firmware_test_manual.md)
