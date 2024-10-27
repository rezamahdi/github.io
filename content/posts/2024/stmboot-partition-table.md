+++
title = "STMBoot: Cryptographic STM32 Bootloader - Partition Table"
date = 2024-10-22

[taxonomies]
tags = ["c", "embedded"]
+++

In [previous](@/posts/2024/stmboot-intro.md) post, I describe the need for a secure bootloader for
STM controllers and explain how it acts. Now, we are going to explain the partition table that the
bootloader uses to provide some functionalities.

<!--more-->

Partitioning the flash is done in order to provide these abilities:
  1. Updating the firmware without losing currently working one, with validating it via
     cryptographic facilities.
  2. Separate data storage form flashing consideration. This lets us to update the firmware without
     overriding the data stored.

As the partition table itself is a critical data, it can't be placed at the end of bootloader
because the bootloader itself can be updated. So it must be placed before it, at the beginning of
executable codes, just after the interrupt table. But there is a problem here. MCU starts the
execution at this address, so there must be executable code at this place. With looking the
traditional MBR structure
([wikipedia](https://en.wikipedia.org/wiki/Master_boot_record#Sector_layout)), We see there is some
code at beginning of partition table. Since this code simply jumps to the real executable, let's
call it *Jumper Code*. In our case, executable is placed just after the partition table, so jumper
code must jump to next address of partition table.

Jumper code is a simple and single `b` instruction in ARM assembly. The address where jumper will
jump is the beginning of actual executable code but there is no such a label to use. So we define
it in linker script:

```ld
  /*The _text lable is defined in linker script as start of codes like this:*/
     .text : {
        *(.vectors)             /* Vector table */
        *(.jumper)              /* Jumper code */
        *(.partition_table)     /* Partition table */

        . = ALIGN(4);
        _text = .;              /* Start of executable code text */
 
        *(.text*)               /* Program code */
     } > flash
```

And jumper code will be as:

```asm
.section .jumper, "x"
  b _text
```

This code yields 2 bytes in Thumb instruction.
