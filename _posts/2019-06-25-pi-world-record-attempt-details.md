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

One of the first things I did was reach out to Alexander Yee, who wrote and maintains a program called y-cruncher[^2]. After emailing back and forth with Alex, I was able to figure out what I would need to achieve my goal: 50 trillion digits of Pi.

I used the y-cruncher program to determine the system requirements which are listed below. For my calculation, the program determined I would need 256TiB of ephemeral storage, and 38TiB for the final output.
![y-cruncher calculation values](/images/y-cruncher-calculation-values.PNG)

One caveat that can trip you up is the fact that y-cruncher calculates everything in tebibytes (base-2), rather than terabytes (base-10). It can get confusing since hard drives are measured in terabytes, and the difference can be significant: for instance, 256TiB is equivalent to 281TB.

Based on the output of y-cruncher, I started looking for hardware on eBay. From February to March of 2019, I started purchasing the equipment that would be required for the attempt: servers, hard drives, various PCI-E cards, a tape libary and drive, to name a few. In the end, this is the hardware I ended up with:

<img src="/images/pi-record-attempt-setup.jpg" alt="record attempt hardware" style="height:800px;"/>

1. HP PROLIANT DL580 Gen8
    * This server (Ubuntu 18.10) performs the Pi computation using the forty-eight attached 6TB hard drives. The final compressed Pi digits are transferred to the Dell R720xd via a cifs-mounted directory (/drives/main_output).
    * (4) Intel Xeon E7-4880V2 2.5GHz 15C/30T CPU
    * 320GB DDR3 PC3-8500R ECC RAM
    * (5) 2.5" 100GB SATA MLC SSD hard drives (RAID 6)
    * (4) 6Gb LSI 9207-8e Dual Port SAS HBA
        * Connected to the four HPE D3600 disk shelves
    * Intel 10-Gigabit X540-AT2 Dual Port Ethernet Adapter
        * Direct connected via (2) CAT6 to Dell R720xd running VEEAM
    * Intel 1-Gigabit I340-T4 Quad Port Ethernet Adapter
        * Used to connect server to the Internet
2. (4) HP StorageWorks D3600
    * These drives are used in lieu of memory to provide y-cruncher the resources required to calculate 50 trillion digits of Pi (mounted on the DL580 as /drives/1 - /drives/48). I originally used HP StorageWorks D2600 disk shelves, but I didn't realize they limited the SATA speed to 3Gb/s. I upgraded to the D3600 disk shelves which support 6Gb/s SATA beginning in June of 2019.
    * (12) HGST Ultrastar He8 HUH728060ALE600 6TB 7.2K 128MB SATA 6Gb 3.5"
3. Dell PowerEdge R720xd
    * This server (Windows Server 2016) serves two primary purposes. First, it runs VEEAM and is connected to a HP tape library. A VEEAM job exists which is run manually monthly to backup the data from the 48 drives. Second, it has additional storage which will store the final compressed output that y-cruncher will generate.
    * (2) Intel Xeon E5-2670 8C/16T CPU
    * 128GB DDR3 ECC RAM
    * (2) 2.5" 600GB SAS 10K hard drives (RAID 1)
    * Emulex LPE 12002, Dual Port 8Gb Fiber Channel HBA
        * Connected to tape library using (2) LC-LC OM3 2m fiber cables
    * Dell PERC H810 Adapter 6Gb RAID Controller Card
        * Connected to Dell MD1200 disk shelf
4. Dell PowerVault MD1200
    * Storage that will hold the final compressed Pi digit output
    * 40TB 7.2K SAS 3.5" (RAID6)
        * (7) HGST Hitachi 4TB
        * (5) Seagate Constellation ES.3 4TB
5. HPE StoreEver MSL4048 Tape Library
    * The tape library is connected to the Dell R720xd via (2) fiber cables. VEEAM is running on the R720xd, and backs up the disk storage on the DL580 directly to tape.
    * LTO-5 FC Tape Drive
    * (21) LTO Ultrium 5 Tape Data Cartridge 1.5TB

One of the first problems I had to address was backing up the ephemeral computational data (48 hard drives). A small subset of the total storage is required to be backed up (around 15TB). y-cruncher generates checkpoint files every so often, and I wrote a Linux bash script to parse the checkpoint file and remove any leftover/unncessary files from being backed up. After that, I have an ansible script which connects to the Windows server running on the Dell R720xd with VEEAM, and it modifies and executes the VEEAM direct file-to-tape backup automatically. The script is also able to provide an estimated completion time, since VEEAM doesn't provide a time estimate natively (it's accurate enough for me at least). I will make these scripts publicly available in the near future.

Here is a rough timeline of the world record attempt (still ongoing):
  * Feb 12, 2019 - This is the day I first reached out to Alex Yee about his y-cruncher program. I told him I was interested in breaking the world record (at that time it was 22.4 trillion digits set by Peter Trueb[^3]). My initial thoughts were to use a NetApp DS4486, NetApp FAS3220 filer, and Dell R910 to compute the record. At this point I wasn't sure how many digits I was going to calculate, but I knew it was going to be at least 22.4 trillion in order to beat the record.
  * Feb 12, 2019 - Alex responds with answers to some questions that I had relating to accessing storage (I originally planned on using iSCSI) and server CPU/memory. I was planning to access the required disk storage using iSCSI on the FAS3220, but Alex pointed out that this would be much too slow even over a 10 Gigabit link. Based on his analysis, I decided to directly connect the storage to the server using SAS cables (SFF-8088). Details were sketchy if I could directly attach the NetApp DS4486 to the main server, or if I would have to use a NetApp filer. After conducting some research, I decided it would just be easier to use external JBOD (non-RAID) disk enclosures. At the time I was looking at using several Dell PowerVault MD1200 enclosures.
  * Feb 13, 2019 - I start looking at the HP Proliant DL580 Gen8 instead of the Dell PowerEdge R910, due to the fact it has a newer processor and higher core count (15/30T vs 10/20T). After some analysis and research, I decide to shoot for 50 trillion digits of Pi. I originally planned on using RAID with the hard drives required during the calculation, but the sheer amount of storage required prevented this possibility. In order to mitigate the risk of a hard drive failure, I decided to utilize a HP MSL4048 tape library, which holds a maximum of 48 tapes. The unit I purchased off of eBay just happened to come with 21 LTO-5 tapes (not mentioned in the auction), so that was an added bonus! I got around to purchasing a single full-height LTO-5 tape drive so that I would be able to backup the hard drives to tape. Around the time I was looking for tape backup solutions, VEEAM was restructuring their licensing, and the community edition now supported tape backups for up to five hosts. Based on this information, I decide to download VEEAM Backup & Replication 9.5 Community Edition and put it on another server connected to the tape library over fiber. After coming to the decision to calculate 50 trillion digits, I determined I would need around 281TB of storage, and after looking on eBay I decided to purchase four HP D2600 external JBOD disk enclosures and forty-eight 6TB SATA drives to meet this need. I also saw that I would need 44TB for the final compressed Pi output. I picked up a Dell MD1200 external JBOD enclosure and twelve 4TB SAS drives that would serve this purpose. While the main 48 drives are not in a RAID volume, the twelve 4TB SAS drives are running in RAID6 to provide drive redundancy.
  * March 14, 2019 - I finally receive all of the hardware necessary for the computation to start. I contact Alex and he recommends I run several tests (I/O performance analysis, swap multiply). I learn that Google has beaten the current world record and was suprised to find that they were interested in this type of computation. Nevertheless, I was still aiming for 50 trillion digits, 18.6 trillion more then they were able to achieve. I also saw that it cost them around $200,000[^4], which seems very expensive for this type of computation. At this point I have still spent less than $10,000.
  * Apr 1, 2019 - All of the testing is completed and the I/O seek values are adjusted to 12Mb/s based on the results (due to drives being limited to 3Gb/s on HP D2600). I start the world record computation.
  * Apr 3, 2019 - I log into the server and I am greeted with the following error message: "A Raid-File operation has failed." After reaching out and troubleshooting, Alex recommends adjusting the ulimit value on Linux (by default, Linux only allows you to have 1023 files handles open by default)[^5]. I set the ulimit value to a very high value in order to prevent this issue from happening again, and this fixes the issue.
  * April 5, 2019  - Status update: Series: S ( 14 ) 2.867%
  * April 29, 2019 - Status update: Series: S ( 7 ) 14.836%
  * May 20, 2019   - Series: S ( 5 ) 23.761%
  * June 15, 2019  - I email Alex asking if there would be any issue upgrading the 3Gb/s SATA D2600 external disk enclosures to the D3600 6Gb/s SATA enclosure. He says there won't be any issue as long as the drives and mount points are the same.
  * June 23, 2019 - I swapped out the D2600 disk enclosures with the D3600 model. I transfer all forty-eight drives to the new enclosures. I had to purchase new drive caddies as well (HP Gen8 model vs Gen5), since the new enclosure drive bays are less wide than the older model. I let Alex know the disk enclosure transfer appears to have been successful and based on the greatly improved transfer speed (it should speed up the computation finish by a month or two) -- write speed: 3.15 GiB/s vs 5.69 GiB/s, read speed: 3.08 GiB/s vs 5.90 GiB/s.
    

[^1]: <https://cloud.google.com/blog/products/compute/calculating-31-4-trillion-digits-of-archimedes-constant-on-google-cloud>
[^2]: <http://www.numberworld.org/y-cruncher/>
[^3]: <http://www.numberworld.org/y-cruncher/news/2016.html#2016_11_15>
[^4]: <https://youtu.be/BwkpNd2ceBk?t=459>
[^5]: <https://easyengine.io/tutorials/linux/increase-open-files-limit/>