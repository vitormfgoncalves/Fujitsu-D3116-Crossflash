# Fujitsu-D3116-Crossflash
How to flash a Fujitsu D3116-C26 RAID card with IT firmware

### Some information first:
The Fujitsu D3116-C26 (RAID) card can be flashed with TI (LSI 2308) firmware to function as an HBA, but its SBR isn’t compatible with the ones available online (ask me how I know). The SBR addresses have a different offset.
For the card to work, it’s necessary to flash a modified SBR with the correct offset. The SBR I’m providing was created from a backup of one of my own cards.

### Before you start:
- Preferably use a non-UEFI system for compatibility reasons. If that’s not possible, search online for how to do it with UEFI. I have some usefull links at the end. PLease do it before you begin.
- **Save the card’s SAS address in a safe place!** Each card has a unique address. **If you lose it, the card will not work again** and you won’t be able to restore it.
- Before performing any destructive action, also **make a backup of the SBR and SPD and store them safely**. You’ll need them if you attempt to revert or recover the card.
- **Make sure your data is safe!** Take your time and check that your backups are up to date. If you're unsure, make an extra backup just in case.
- Make sure you understand how to change the BIOS (or UEFI) settings of your motherboard and you are confortable doing it. You may need to change the boot order.
- The `Sources` folder is provided for archival purposes only. It contains the files I used during the process. Original filenames have been preserved to make them easier to search for online.
- You should have another computer available for other tasks and research while your card is being flashed.
- Prepare the USB stick (FreeDOS 1.3) in advance with all the necessary files. See bellow.
- In FreeDOS, always use option 4 (Safe Mode). Some tools won’t work otherwise.
- `megarec` is the recovery tool, and if it doesn’t detect the card, there’s nothing more you can do — it’s dead.
- `sas2flash` (or `sas2flsh`) is the tool used to flash the firmware. The name may differ slightly, but it’s the same tool. Sometimes, specific versions are required — be prepared to do a bit of research, especially when using UEFI.
- The backup module (battery) is no longer needed and has no function with the IT firmware. I recommend removing it. Reinstall it if you revert to the RAID firmware.

# Preparation:
- Create a FreeDOS 1.3 USB boot drive ([https://www.ibiblio.org/](https://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/distributions/1.3/official/)) with Rufus (I used the "Full" version)
- Clone this repository, or download it and extract all files to a folder.
- Copy the `LSI` folder to the USB drive root. Also copy the `efi` folder [if you going the UEFI route](https://github.com/tianocore/edk/tree/master/Other/Maintained/Application/UefiShell/bin/x64).
- Disconnect all disks (even NVMe, if any)
- Boot to FreeDOS using the USB drive

# Procedure to install the firmware:

1. After booting into FreeDOS using option 4 (Safe Mode), navigate to the folder containing the files (`cd LSI`) and check if the card is detected:

   ```bash
   $ megarec -adpList
   ```

2. Back up the SBR and SPD (needed if you want to restore the card later):

   ```bash
   $ megarec -readsbr 0 BAK2208.SBR
   $ megarec -readspd 0 BAK2208.SPD
   ```

3. Note the card's SAS address **(VERY IMPORTANT!)**:

   - Save it to a file:
     ```bash
     $ megacli -adpAllInfo -a0 > FJ2208.TXT
     ```
   - And/or display it on screen (you can take a photo):
     ```bash
     $ megacli -adpAllInfo -a0 | more
     ```

4. **After making sure you’ve backed up the SBR and correctly noted the SAS address**, write a blank SBR to the card:

   ```bash
   $ megarec -writesbr 0 empty.sbr
   ```

5. Clear the card’s flash memory (may take a few moments):

   ```bash
   $ megarec -cleanflash 0
   ```

6. Reboot the computer, boot FreeDOS again with option 4 (Safe Mode), and return to the folder.

7. At this point, the card will not be detected by `sas2flash`. You must write the modified SBR:

   ```bash
   $ megarec -writesbr 0 FJ2208M.SBR
   ```

8. Repeat step 6.

9. Check if the card is detected by `sas2flash`. You may ignore the error about the firmware not being operational (just press Enter):

   ```bash
   $ sas2flsh -list
   ```

10. Flash the new firmware to the card (`mptsas2.rom` is not strictly required, but recommended). This may take a few minutes:

    ```bash
    $ sas2flsh -o -f 2308T207.ROM -b mptsas2.rom
    ```
    **Note:** If the card gets stuck on “Initializing...”, restart the process and, in this step, flash the firmware without the BIOS: `sas2flsh -o -f 2308T207.ROM`. The BIOS is only required if you intend to boot the system from one of the drives connected to the card. Once inside the OS, it will work normally.

11. Restore the SAS address (previously saved):

    ```bash
    $ sas2flsh -o -sasadd 5003005701XXXXXX
    ```

12. Reboot and verify everything is working as expected.

# Procedure to Restore the Card

This procedure can also be used if something goes wrong. It is important that you have the backup files from the card.

1. After booting into FreeDOS using option 4 (Safe Mode), navigate to the folder containing the files (`C:\LSI`) and check if the card is detected:

   ```bash
   $ megarec -adpList
   ```
2. Clear the card's SBR:
   ```bash
   $ megarec -writesbr 0 empty.sbr
   ```
3. Clear the card’s flash memory:
   ```bash
   $ megarec -cleanflash 0
   ```
4. Reboot the computer, boot FreeDOS again with option 4 (Safe Mode), and return to the folder.
5. Restore the SBR from backup:
   ```bash
   $ megarec -writesbr 0 BAK2208.SBR
   ```
6. Write the recovery firmware. This step may take several minutes:
   ```bash
   $ megarec -m0flash 0 2208_16.rom
   ```
7. Reboot and verify everything works as expected. The card will have no configuration, but should be fully operational.
   If you removed the battery module, it will show a warning during boot. Just put it back and the problem should be solved.

# Final notes:

These cards are quite resilient. **As long as you have the SAS address** and a backup of the SBR, it’s almost impossible to brick them. Even without any firmware installed, the megarec tool will almost certainly still detect the card.
However, **I am not responsible for what may happen to your card.** Before proceeding, make sure the card is not essential to the safety of your data, and ensure you have a backup in case something goes horribly wrong.

**USE AT YOUR OWN RISK!**

### Usefull online resouces:
- [https://marcan.st/2016/05/crossflashing-the-fujitsu-d2607/](https://marcan.st/2016/05/crossflashing-the-fujitsu-d2607/)
- [https://web.archive.org/web/20250322053307/https://mywiredhouse.net/blog/flashing-lsi-2208-firmware-use-hba/](https://web.archive.org/web/20250322053307/https://mywiredhouse.net/blog/flashing-lsi-2208-firmware-use-hba/)
- [https://www.truenas.com/community/threads/how-to-flashing-lsi-sas-hba-controller-efi-uefi.78457/](https://www.truenas.com/community/threads/how-to-flashing-lsi-sas-hba-controller-efi-uefi.78457/)
- [https://www.truenas.com/community/threads/flashing-the-lsi2308-firmware-on-a-supermicro-x10sl7-f-motherboard.38884/](https://www.truenas.com/community/threads/flashing-the-lsi2308-firmware-on-a-supermicro-x10sl7-f-motherboard.38884/)
- [https://www.truenas.com/community/threads/page-fault-when-using-megarec-exe-and-freedos-1-1.41129/](https://www.truenas.com/community/threads/page-fault-when-using-megarec-exe-and-freedos-1-1.41129/)
- [https://github.com/marcan/lsirec](https://github.com/marcan/lsirec)
- [https://github.com/marcan/lsirec/issues/1](https://github.com/marcan/lsirec/issues/1)
- [https://github.com/maximko/SAS2208-to-SAS2308-IT-X9DRH-7F](https://github.com/maximko/SAS2208-to-SAS2308-IT-X9DRH-7F)
- [https://forums.serverbuilds.net/t/flashing-sas2208-to-it-mode-when-sas2flsh-does-not-detect-it/2357](https://forums.serverbuilds.net/t/flashing-sas2208-to-it-mode-when-sas2flsh-does-not-detect-it/2357)
- [https://forums.servethehome.com/index.php?threads/is-there-a-way-to-restore-an-lsi-2208-after-firmware-update-failure.13237/](https://forums.servethehome.com/index.php?threads/is-there-a-way-to-restore-an-lsi-2208-after-firmware-update-failure.13237/)
- [https://github.com/HomeofNever/Blog/tree/master/pages/_assets/file/lsi-9266-8i-to-it-mode](https://github.com/HomeofNever/Blog/tree/master/pages/_assets/file/lsi-9266-8i-to-it-mode)
- [https://blog.grem.de/sysadmin/LSI-SAS2008-Flashing-2012-04-12-22-17.html](https://blog.grem.de/sysadmin/LSI-SAS2008-Flashing-2012-04-12-22-17.html)
- [https://lazymocha.com/blog/2020/06/05/cross-flash-ibm-servraid-m5110-and-h1110-to-it-hba-mode/](https://lazymocha.com/blog/2020/06/05/cross-flash-ibm-servraid-m5110-and-h1110-to-it-hba-mode/)
- [https://github.com/tianocore/edk/tree/master/Other/Maintained/Application/UefiShell/bin/x64](https://github.com/tianocore/edk/tree/master/Other/Maintained/Application/UefiShell/bin/x64)

##### Good Luck!
*This repository will not receive updates. I won’t respond to issue reports. It’s intended solely as a record of my experience with these cards, which, despite their age, remain very useful.*
