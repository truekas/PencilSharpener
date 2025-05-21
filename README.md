
<img src="https://github.com/truekas/PencilSharpener/blob/main/src/Logo.png?raw=true" alt="Pencil Sharpener"/>

> [!WARNING]  
> DURING THE EXPLOIT, MAKE SURE THE CHROMEBOOK STAYS PLUGGED INTO A CHARGER AT ALL TIMES (EXCEPT WHILE BRIDGING THE WP PINS). THE SYSTEM WILL BRICK IF THIS IS NOT FOLLOWED.

> [!IMPORTANT]
> This exploit is for *non-factory keyrolled* Ti50 systems only, meaning that your Chromebook had its *shim keys changed in an update* and not during its assembly. We are not responsible for any damage done to your organization or your device. This writeup is for educational purposes only.

## Hey There!
Google will release changes in version 136 (scheduled for May 13, 2025) that will prevent the use of Pencil Sharpener and may damage your device if you attempt to use this exploit. If you are reading this after the update, we suggest you find another method to unenroll your device.

If you are an administrator, we recommend that you set `DeviceMinimumVersion` in the admin console to ensure that your Chromebooks get updated as soon as v136 is available. 

## Introduction 
The Pencil Sharpener exploit enables users to unenroll *non-factory-keyrolled* Ti50 Chromebooks using a modified version of the pencil method. This exploit works because the Google Security Chip (GSC) does not verify devices hashes until the device loses power, allowing users to disable RO verification before anything is checked.

You can watch our proof of concept video on Odysee if you need a more visual demonstration of the exploit:
<br>
[![Video Demo](https://github.com/truekas/PencilSharpener/blob/main/src/Cover.png?raw=true)](https://ody.sh/xySDCFhvHi)

**Special Thanks To:**
- FCPS DIT :D        | Letting us post
- CoolElectronics    | Original Pencil Method
- Kelsea             | Miku Energy Drink
- Appleflyer         | Blog helped improve Ch341a attachment instructions
- Mercury Workshop   | Sh1mmer 
  
## The Exploit
**The materials you need:** (we reccommend [this kit](https://www.amazon.com/AiTrip-EEPROM-Programmer-CH341A-Adapter/dp/B07VNVVXW6) for SOIC-8 chips and [this one](https://a.co/d/5dswJ8O) for WSON-8)

- SOIC-8 or WSON-8 chip clip
- Paperclip or Safety Pin
- Linux system with flashrom installed
- A screwdriver (duh)
- Sh1mmer image for your board
- Working charger
- Ch341a Flash Programmer
- A ChromeOS recovery USB for v124
- Lots of time and a few brain cells

First, fully power off and unplug your device, flip it over, and open the back to access the mainboard.

> [!TIP] 
> **How to Setup Your Chip clip:**
> \
> \
> Take your chip clip, and a safety pin (recommended) or paperclip. If you are using a safety pin, cut off the bigger side.\
> Put the paperclip or safety pin into holes 3 and 8. To find these, find the red wire. This is pin 1. From there, you can find the other pins.
> With pin 1 in the top left, pin 3 would be the 3rd pin in the top row, and pin 8 would be directly under pin 1.

Then, disconnect the battery from the mainboard and locate your Flash Chip. Bridge pins 3 and 8 with your conductive material or chip clip.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/2.png?raw=true" alt="2.png"/>

Flip your laptop to its side with the charging port facing up and the pins still bridged. Plug in your device and push `esc + refresh + power` to enter the device recovery menu, then press `ctrl + d`. As soon as the screen goes black, press the keys to re-open the recovery menu. 

Insert your Sh1mmer USB and then choose to boot from it. You may get a `no valid image` error. If this happens, you need to re-flash the correct keys to the device using instructions in the [rolled keys](#fixing-rolled-keys) section.

After booting into Sh1mmer, you should choose `utilities > unenroll`. After it gives an error, open the bash console WHILE MAKING SURE THE PINS ARE STILL BRIDGED and run:
```
flashrom --wp-disable
/usr/share/vboot/bin/set_gbb_flags.sh 0x80b3
flashrom --wp-enable
```
> [!IMPORTANT]
> On newer versions, if you get an error saying "owner has disabled downgrading" or "verified images only" you must go into sh1mmer and turn off rootfs verification.

Hit `esc + refresh + power` to return to the recovery menu and boot onto your v124 recovery USB. If you have any issues before or after the recovery process, follow the [rolled keys steps](#fixing-rolled-keys).

After the recovery is complete, boot into ChromeOS. Then, on the sign-in screen switch to VT2 by pressing `ctrl + alt + f2`. 

If you are prompted to login on the console, try to log in as `chronos` with no password, and elevate to root by using `sudo -i`. If that does not work, you can try logging in as `root` and then using `test0000` as your password. After you have access to the shell, run the following commands:
```
tpm_manager_client take_ownership
cryptohome --action=remove_firmware_management_parameters
crossystem dev_boot_usb=1
```

Reconnect the battery to the motherboard, and run `gsctool -a -o`. Follow the prompts to push the power button and the system should automatically reboot. When the device turns back on, re-open the recovery menu and re-enable devmode.

Afterward, go to the VT2 console, run `gsctool -a -I AllowUnverifiedRo:always`, and the device should be unenrolled.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/unenrolled.png?raw=true" alt="unenrolled"/>

## Fixing Rolled Keys
**IMPORTANT: THIS WILL NOT UNROLL FACTORY ROLLED KEYS!!**
During the downgrade process or while trying to use Sh1mmer, you may be unable to boot onto the devices. This is because your system's shim keys have been changed in an update. 

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/rolledkeys.png?raw=true" alt="ch341a and shell"/>

This issue is fixed by re-flashing the correct keys to the system. Here's how to do it:

First, take your ch341a flash programmer and attach it to your chip clip (the red wire connects to number 1 on the ch341a). Take the end of your chip clip, and re-attach it to your flash chip. Now [connect](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html#prepping-to-flash) using your Linux system and run the following commands: 

If you are not using a flash programmer, remove `-p ch341a_spi` from the commands you run.

```bash
flashrom --wp-disable
futility gbb -p ch341a_spi -r file.bin #extract keys as file.bin
futility gbb -p ch341a_spi -s -r file.bin
flashrom --wp-enable
```

If you are using **a Nissa board**, you can use our working example with the correct keys already extracted. You should still remove `-p ch341a_spi` if you aren't using a programmer. 

```bash
flashrom --wp-disable
curl -O https://github.com/truekas/PencilSharpener/raw/refs/heads/main/src/nissa_keys.bin
futility gbb -p ch341a_spi -s -r nissa_keys.bin
flashrom --wp-enable
```

## Re-Enrolling
You can re-enroll your device by accessing a VT2 shell, typing `vpd -i RW_VPD -s check_enrollment=1`, and then powerwashing the device using `CTRL + ALT + SHIFT + R`.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/enrolled.png?raw=true" alt="enrolled screen"/>

# Works Cited

Appleflyer. *"A Simple Guide to Flash WSON-8/SOIC-8 Chromebook Chips."* appleflyer's blog, edited by Appleflyer,  
14 Apr. 2024, [appleflyers-blog.vercel.app](https://appleflyers-blog.vercel.app/blog/gbbflagflash). Accessed 12 Dec. 2024.

Banana, Bin Bash. *"GBB flag-inator."* Edited by Kkilobyte, Bin Bash Banana, revision 1cbafcb,  
4 Sept. 2024, [GitHub](https://github.com/binbashbanana/gbbflaginator/). Accessed 2 Dec. 2024.

*"Firmware Boot and Recovery."* *The Chromium Projects*, Google,  
[Chromium Docs](https://www.chromium.org/chromium-os/chromiumos-design-docs/firmware-boot-and-recovery/). Accessed 26 Nov. 2024.

*"Firmware Management Parameters."* *Firmware Management Parameters*, Google,  
[Chromium Docs](https://www.chromium.org/chromium-os/fwmp/). Accessed 26 Nov. 2024.

*"Futility GBB Documentation."* *Google Git*, version 1.0, Google,  
[Source](https://chromium.googlesource.com/chromiumos/platform/vboot_reference/+/refs/heads/main/futility/docs/cmd_gbb_utility.md). Accessed 12 Dec. 2024.

Hughes, Tom. *"Embedded Controller (EC)."* Edited by Keith Short et al., *CrOS EC (Embedded Controller)*,  
revision 85d7598, Google, 10 Apr. 2024,  
[Source](https://chromium.googlesource.com/chromiumos/platform/ec/+/HEAD/README.md). Accessed 8 Dec. 2024.

MrChromebox. *"Disabling Firmware Write Protection."* *MrChromebox.tech*, edited by MrChromebox et al.,  
revision 9a03f14, 8 Nov. 2024, [Docs](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html). Accessed 26 Nov. 2024.

---. *"Firmware Write Protection on ChromeOS Devices."* *MrChromebox.tech*, revision e219c93,  
5 July 2024, [Docs](https://docs.mrchromebox.tech/docs/firmware/wp/). Accessed 26 Nov. 2024.

Olyb, et al. *"Breaking ChromeOS's Enrollment Security Model: A Postmortem."* *coolelectronic blog*,  
[Blog](https://blog.coolelectronics.me/breaking-cros-6/). Accessed 26 Nov. 2024.

*"Read-only firmware unlock on 2023+ devices."* *The Chromium Projects*, Google,  
[Chromium Docs](https://www.chromium.org/chromium-os/developer-library/guides/device/ro-firmware-unlock/). Accessed 26 Nov. 2024.

Rink, Jett. *"Google Security Chip (GSC) Case Closed Debugging (CCD)."* *Google Git*, version f5e2cf9, Google,  
24 Apr. 2024,  
[Source](https://chromium.googlesource.com/chromiumos/platform/ec/+/cr50_stab/docs/case_closed_debugging_gsc.md). Accessed 26 Nov. 2024.

TerAvest, Justin. *"Firmware Test Manual."* Edited by Mike Frysinger et al., *Chromium OS Docs*, revision 05bebd5, Google,  
27 Jan. 2021,  
[Source](https://chromium.googlesource.com/chromiumos/docs/+blame/master/firmware_test_manual.md). Accessed 12 Dec. 2024.

*"Unbricking/Flashing with a ch341a USB programmer."* *Chrultrabook Docs*, 29 Jan. 2025,  
[Source](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html). Accessed 15 Apr. 2025.

*"Verified Boot."* *The Chromium Projects*, Google,  
[Chromium Docs](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot/). Accessed 26 Nov. 2024.

*"Verified Boot Data Structures."* *The Chromium Projects*, Google,  
[Chromium Docs](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot-data-structures/). Accessed 26 Nov. 2024.
