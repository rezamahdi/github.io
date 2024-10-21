+++
title = "STMBoot: Cryptographic STM32 Bootloader - Introduction"
date = 2024-10-09

[taxonomies]
tags = ["c", "embedded"]
+++

Microcontrollers are the native processing power in industries. As you may know, the difference
between microcontroller and microprocessor is the ability of providing industrial needs versuse
high performance capabilities.

Afterward, As I really make a noisy and histeric fouces on security, allways I want to make secure
embedded projects that have some secure facilities for bouth user and origin. The first thing that
made me intrested is the bootloader.

<!-- more -->
{{mermaid_import() }}

Bootloader must provide some sort of security for origin and user. For user, It must keep the data
stored in flash secure, don't allow unknown parties to flash the device and make upgrading easy and
rollbackable. For origin, it must keep the firmware secure from any other parties (including user)
by rejecting reading flash.

So there must be a way to authenticate the party that attempts to do any work with the device.
Best method that I found is using Elliptic Curve Cryptography (ECC) as it is fast, secure and has
short key size.

{% mermaid(float="left") %}
---
title: Boot sequence diagram
---
graph TD
    A[Device Boot & Boot loader runs] --> C{Should trap?}
    C --YES--> D[Wait for communication]
    D --> G[Authenticate]
    G -->I[Run utility]
    G --> H[Reject and boot]
    H --> E
    C --NO--> E(Find 'Current' firmware partition and boot)
{% end %}

The procedure is as this: bootloader will check for need of booting the firmware or trap in flashing
utility. If the party wants to do something with device, must be authenticated by keys that is stored
in the flash. Just the public keys are stored so reading this keys may not make any security flaws.
After authentication, commands can be run with some sort of protocols like Hayes (at commands)
or df2.

# Partition table
For simple management of firmware and the data, flash must be partitioned. So there is a partition table 
at the beggining of flash. At least there is two bootable partitions, one for keeping the current
firmware, and other one for upgrading. If validating new firmware is successful, the current firmware
will be set as the partition of update one.

As firmware may use flash to store some data, there must be additional partitions for storing this data.
Something that is critical about the partitions (and even bootloader itself) is the size of partition.
I assume the typical flash space of chips to be 128KB. Some space is occupied by bootloader and partition
table. If in total keep the size as 32KB, 96KB remains for partitions.
<br/>

# Parts
1. Intorduction
2. [Partition Table](@/posts/2024/stmboot-intro.md)
3. [Authentication and party roles](@/posts/2024/stmboot-intro.md)
