---
title: Kekboot
date: 2025-05-11 21:49:24
category: code
permalink: /coding/kekboot/
description: "Boot option by Wake-up Type"
---

[View Code on Github.](https://github.com/KastenKlicker/Kekboot) <a class="icon u-url" target="_blank" rel="noopener me" href="https://github.com/KastenKlicker/Kekboot" aria-label="github" title="github"><i class="fa-brands fa-github"></i></a>

## Introduction
Kekboot is a bootloader designed to chainload other EFI files, like an OS, depending on the systems Wake up type.

## Using Kekboot
### How to build
1. Clone the [GNU EFI](https://sourceforge.net/projects/gnu-efi/) repository and follow the instructions the README.gnuefi.
2. Test if one of the included GNU EFI apps is working
3. Clone this repository into the apps directory
4. Build the EFI application, like stated in the README.gnuefi, it's inevitable to read the README.gnuefi!


### How to Use

1. **Create a Data File**

    Create a UTF-16 encoded file named `myvar.data`.  
    It should contain **9 entries** in the following format (all on one line):

    `partitionGUID=filePath partitionGUID=filePath ...`

    - `partitionGUID`: The GUID of the partition where the chainload EFI file is located.  
    - `filePath`: The path to the EFI file on that partition.  
    - The **position** of entries determines the **WakeUpType**.
    | Position | WakeUpType           |
    | -------- | -------------------- |
    | 0        | Reserved             |
    | 1        | Other                |
    | 2        | Unknown              |
    | 3        | APM TIMER            |
    | 4        | Modem Ring           |
    | 5        | LAN Remote           |
    | 6        | Power Switch         |
    | 7        | PCI PME#             |
    | 8        | AC Power Restored    |

    > 💡 You can find the `partitionGUID` and `filePath` using the `efibootmgr` command.

    2. **Write the EFI Variable**  
    Use the `efivar` command to write the variable:  
    `efivar --write --name=1be4df61-93ca-11d2-aa0d-00e098032b8c-WakeUpType --datafile=myvar.data`


    3. **Verify the Variable**  
    Check that the variable was written correctly:  
    `efivar --print --name=1be4df61-93ca-11d2-aa0d-00e098032b8c-WakeUpType`

    4. **Prepare the EFI Application**  
    Rename your built EFI application to: **BOOTX64.EFI**
    Then copy it to the **EFI system partition**.

    5. **Reboot**  
    Restart your system and boot from the EFI partition.

## How it works

Kekboot utilizes the SMBIOS (System Management BIOS) specification to retrieve the WakeUpType.  
It does this by iterating through the SMBIOS tables provided by UEFI, searching for the **System Information** table (Type 1) and the Wake-up Type byte.

If found, the EFI application proceeds to read and parse the previously created EFI variable.

Finally, the EFI file corresponding to the Wake-up Type is located on the appropriate partition, loaded into main memory, and prepared for execution.


