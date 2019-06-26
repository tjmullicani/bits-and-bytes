---
title: "Calculating Pi: My attempt at breaking the Pi World Record"
excerpt_separator: "<!--more-->"
author: Timothy Mullican
permalink: calculating-pi-my-attempt-breaking-pi-record
comments: false
categories:
  - Pi
tags:
  - Pi
  - Math
  - World Record
date: 2019-06-25T20:38:39-05:00
---

Back in February of 2019, I had the bright idea to break the world record for Pi. My primary reason for attempting to break the record set by Emma Haruka Iwao/Google[^1]\ (March, 2019 -- 31.4 trillion digits) is to test the upper limits of my server hardware, as well as prove that you don't need to spend hundreds of thousands of dollars to achieve something great.

[^1]: <https://cloud.google.com/blog/products/compute/calculating-31-4-trillion-digits-of-archimedes-constant-on-google-cloud>

One of the first things I did was reach out to Alexander Yee, who wrote and maintains a program called y-cruncher[^2]. After emailing back and forth with Alex, I was able to figure out what I would need to achieve my goal: 50 trillion digits of Pi.

[^2]: <http://www.numberworld.org/y-cruncher/>

I used the y-cruncher program to determine the system requirements which are listed below. For my calculation, the program determined I would need 256TiB of ephemeral storage, and 38TiB for the final output.
![y-cruncher calculation values](/images/y-cruncher-calculation-values.PNG)

One caveat that can trip you up is the fact that y-cruncher calculates everything in tebibytes (base-2), rather than terabytes (base-10). It can get confusing since hard drives are measured in terabytes, and the difference can be significant: for instance, 256TiB is equivalent to 281TB.

Based on the output of y-cruncher, I started looking for hardware on eBay. From February to March of 2019, I started purchasing the equipment that would be required for the attempt: servers, hard drives, various PCI-E cards, a tape libary and drive, to name a few. In the end, this is the hardware I ended up with:
1. HP PROLIANT DL580 Gen8
  1. (4) Intel Xeon E7-4880V2 2.5GHz 15C/30T CPU
  2. 320GB DDR3 PC3-8500R ECC RAM
      (8) 8GiB
      (16) 16GiB
  3. (5) 2.5" 100GB SATA MLC SSD hard drives (RAID 6)
  4. (4) 6Gb Dual Port SAS HBA
    * Connect to the four HPE D3600 disk shelves
    1. (3) LSI 9207-8e
    2. (1) Dell PERC H200e (had it lying around...)
  5. (1) Intel 10-Gigabit X540-AT2 Dual Port Ethernet Adapter
    * Direct connected via (2) CAT6 to Dell R720xd running VEEAM
  6. (1) I340-T4 Gigabit Quad Port Ethernet Adapter
    * Used to connect server to the Internet
2. (4) HP StorageWorks D3600
  * These drives are used in lieu of memory to provide y-cruncher the resources required to calculate 50 trillion digits of Pi. I originally used HP StorageWorks D2600 disk shelves, but I didn't realize they limited the SATA speed to 3Gb/s. I upgraded to the D3600 disk shelves which support 6Gb/s SATA beginning in June of 2019.
  1. (12) HGST Ultrastar He8 HUH728060ALE600 6TB 7.2K 128MB SATA 6Gb 3.5"
3. Dell PowerEdge R720xd
  * This server serves two primary purposes. First, it runs VEEAM and is connected to a HP tape library. A VEEAM job exists which is run manually monthly to backup the data from the 48 drives. Second, it has additional storage which will store the final compressed output that y-cruncher will generate.
  1. (2) Intel Xeon E5-2670 8C/16T CPU
  2. 128GB DDR3 ECC RAM
    1. (8) 16GiB
  3. (2) 2.5" 600GB SAS 10K hard drives (RAID 1)
  4. (1) Emulex LPE 12002, Dual Port 8Gb Fiber Channel HBA
    * Connected to tape library using (2) LC-LC OM3 2m fiber cables
  5. Dell PERC H810 Adapter 6Gb RAID Controller Card
    * Connected to Dell MD1200 disk shelf
4. Dell PowerVault MD1200
  * Storage that will hold the final compressed Pi digit output
  1. 40TB 7.2K SAS 3.5" (RAID6)
    1. (7) HGST Hitachi 4TB
    2. (5) Seagate Constellation ES.3
5. HPE StoreEver MSL4048 Tape Library
  * The tape library is connected to the Dell R720xd via (2) fiber cables. VEEAM is running on the R720xd, and backs up the disk storage on the DL580 directly to tape.
  1. (1) LTO-5 FC Tape Drive
  2. (21) LTO Ultrium 5 Tape Data Cartridge 1.5TB

Here is a rough timeline of the world record attempt (still ongoing):
  * Feb 12, 2019 - This is the day I first reached out to Alex Yee about his y-cruncher program. I told him I was interested in breaking the world record (at that time it was 22.4 trillion digits set by Peter Trueb[^3])

[^3]: <http://www.numberworld.org/y-cruncher/news/2016.html#2016_11_15>