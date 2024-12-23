
<img src="https://github.com/truekas/PencilSharpener/blob/main/src/Logo.png?raw=true" alt="Pencil Sharpener"/>

> [!WARNING]  
> DURING THE EXPLOIT, MAKE SURE THE CHROMEBOOK STAYS PLUGGED IN TO A CHARGER AT ALL TIMES (EXCEPT WHILE BRIDGING THE WP PINS). THE SYSTEM WILL BRICK IF THIS IS NOT FOLLOWED.

> [!IMPORTANT]
> This writeup is for educational purposes only. Do not use this exploit on your organization's systems without permission. Remember that your orginization's Chromebook is not your personal device. Nobody is responsible for any touble that happens because of this exploit.

## Hey There!
If you're looking at this writeup, this exploit has been patched. Recently, Google pushed a security update that detects changes to the write-protected (read-only) portion of the Chromebook's firmware. This prevents the use of this exploit on up to-date systems. We decided to release this writeup after the patch, to ensure that no damage is caused to the devices of schools and companies. 

If you are an administrator, we recommend that you set the `DeviceMinimumVersion` in Google Admin to ensure that all new Chromebooks have been patched. We also recommend that you check the policy sync dates for all users on Cr50 Chromebooks to monitor unenrollment through other exploits.

## Introduction 
This writeup demonstrates how Google's Fog patch can be bypassed on Ti50 systems. The exploit uses a modified version of the original pencil method to unenroll the system and prevent it from bricking.

**Special Thanks To:**
- Fairfax County Public Schools Deparment of Information Technology :D 
- Appleflyer | who's blog post helped us to write improved instructions on setting up the ch341a
- Kilobyte | Helping with re-enrollment process
- Kelpstream | Gave continuous feedback and helped test Pencil Sharpener
  
## The Exploit
**What you need:**
- Chip Clip
- Paperclip or Safety Pin
- A Linux system with flashrom installed
- A screwdriver (duh)
- Sh1mmer image for your device
- Working charger
- Ch341a Flash Programmer
- A ChromeOS recovery USB for v124
- A few braincells
- Lots of time

First, fully power off and unplug your device, flip it over, and open the back to gain access to the mainboard.

> [!TIP] 
> **How to Setup Your Chip clip:**
> \
> \
> Take your chip clip, and a safety pin (recommended) or paperclip. If you are using a safety pin, cut off the bigger side.\
> Put the paperclip or safety pin into holes 3 and 8. To find these, find the red wire. This is pin 1. From there, you can find the other pins.
> With pin 1 in the top left, pin 3 would be the 3rd pin in the top row, and pin 8 would be directly under pin 1.

Then, disconnect the battery from the mainboard and locate your device's Flash Chip. Bridge pins 3 and 8 with your conductive material or chip clip.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/2.png?raw=true" alt="2.png"/>

Afterward, re-insert your charger AND KEEP IT PLUGGED IN while pushing `esc + refresh + power` to enter the device recovery menu. Then press `ctrl + d`. As soon as the screen goes black, press the keys to re-open the recovery menu. 

Insert your Sh1mmer USB and then choose to boot from it. You may get a `no valid image` error. If this happens, you need to re-flash the correct keys to the device using instructions in the [rolled keys](#fixing-rolled-keys) section.

Re-open the recovery menu and boot into Sh1mmer. You should choose `utilities > unenroll` and after it gives an error, open the bash console WHILE MAKING SURE THE PINS ARE STILL BRIDGED and run:
```
flashrom --wp-disable
/usr/share/vboot/bin/set_gbb_flags.sh 0x80b3
flashrom --wp-enable
```
Hit `esc + refresh + power` to go back into the recovery menu and boot onto your v124 recovery USB. Follow the [rolled keys steps](#fixing-rolled-keys) if you keyroll again.

After the recovery process is complete, choose to boot into ChromeOS. Then switch to the VT2 console on the sign-in screen by pressing `ctrl + alt + f2`. 

If you are prompted to login on the console, try to login as `chronos` with no password, and elevate to root by using `sudo -i`. If that does not work, you can also try logging in as `root` and then using `test0000` as your password. After you have access to the shell, run the following commands:
```
tpm_manager_client take_ownership
cryptohome --action=remove_firmware_management_parameters
crossystem dev_boot_usb=1
```

Make sure that the battery is re-inserted on the mainboard and run `gsctool -a -o`. Follow the prompts to push the power button and the system should automatically reboot. When the device turns back on, re-open the recovery menu and re-enable devmode.

Afterward, go back to the VT2 console, run `gsctool -a -I AllowUnverifiedRo:always`, and the device should be unenrolled.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/unenrolled.png?raw=true" alt="unenrolled"/>

## Fixing Rolled Keys
**IMPORTANT: THIS WILL NOT UNROLL FACTORY ROLLED KEYS!!**
During the downgrade process or while trying to use Sh1mmer you may be unable to boot onto the media. This is because your systems shim keys have been rolled in an update. 

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/rolledkeys.png?raw=true" alt="ch341a and shell"/>

This issue is fixable by flashing the correct keys to the system. Here's how to do it:

First, take your ch341a flash programmer and attach it to your chip clip (the red wire connects to number 1 on the ch341a). Take the end of your chip clip, and re-attach it to your flash chip. Now [connect to your device](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html#prepping-to-flash) though your linux system and run the following commands: (this can also technically be done through VT2)

If you are not using a flash programmer, remove `-p ch341a_spi` from the commands you run.

```bash
flashrom --wp-disable
futility gbb -p ch341a_spi -r file.bin #extract keys as file.bin
futility gbb -p ch341a_spi -s -r file.bin
flashrom --wp-enable
```

If you are using a device **with a Nissa board**, you can use our working example with the correct keys already extracted. You should still remove `-p ch341a_spi` if you are not using a programmer. 

```bash
flashrom --wp-disable
curl -O https://github.com/truekas/PencilSharpener/raw/refs/heads/main/src/nissa_keys.bin
futility gbb -p ch341a_spi -s -r nissa_keys.bin
flashrom --wp-enable
```

## Re-Enrolling
It is possible to re-enroll your device by accessing a VT2 shell, typing `vpd -i RW_VPD -s check_enrollment=1`, and then powerwashing the device using `CTRL + ALT + SHIFT + R`.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/enrolled.png?raw=true" alt="enrolled screen"/>

## Citations (Not MLA)
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
