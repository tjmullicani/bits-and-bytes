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
last_modified_at: 2019-06-27T00:45:08-05:00
---

Before I go into the specifics of my record attempt, it is important for you to have some background information. Over the past few years I have been steadily acquiring server hardware to increase my knowledge of system administration and cyber security. This is not only for personal growth: it has also helped me immensely in my professional career. I have served in various roles including cyber security (Cyber Defense Analyst, Chief Information Security Officer) and system administration (Senior System Administrator). Historically, I have also donated my spare system computing resources to various distributed computing projects through the BOINC[^1] platform. In late 2018 I decided that I shift the primary focus of my lab to distributed computing. To support this goal I acquired the HP c7000 bladesystem that you will see later in the photo of my homelab. Anyways, back to the reason you are here: My primary reason for attempting to break the record set by Emma Haruka Iwao/Google[^2]  (March, 2019 -- 31.4 trillion digits) is to test the upper bounds of my server hardware. Additionally I wanted to prove that people, rather than businesses with vast pools of resources and money, can still do achieve marvelous things.

One of the first things I did was reach out to Alexander Yee, who wrote and maintains a program called y-cruncher[^3]. After emailing back and forth with Alex, I began to determine what would be required for such a feat: 50 trillion digits of Pi. In order to calculate this many digits, y-cruncher determined I would need 256TiB of ephemeral storage, in addition to 38TiB for the final output.
![y-cruncher calculation values](/images/y-cruncher-calculation-values.PNG)

One of the first things that you may notice is that y-cruncher calculates everything in tebibytes (base-2), rather than terabytes (base-10). It can get confusing since hard drives are measured in terabytes, and the difference can be significant: for instance, 256TiB is equivalent to 281TB. You don't want to get 3/4 of the way through a record attempt and realize you don't have enough storage.

The next caveat I was warned about is that y-cruncher will leave various files on the hard drives that are not required to be backed up. In order to ensure the checkpoint file and files on the disk are not overwritten, you must first stop the y-cruncher program before starting a backup. For instance, imagine the following scenario:
1. After y-cruncher has been running for a month, you issue a Ctrl-C to cancel the program. If you were to examine the various hard drives used in the computation, you would see a bunch of files. However, not all of these are needed. In order to determine which ones are actually required for the computation, you must first parse the y-cruncher checkpoint file.
2. For me, since I had limited space in my tape library, I didn't want to backup erroneous files that were not required to restore in the event of a failure. Plus, the additional erroneous files could eventually fill up the disks and cause them to run out of space. It was easier to just write a simple Bash script to delete the files that were not required.

Enough about the various issues you must keep in mind, though. Once I had a rough estimate of the amount of equipment and disk required for the Pi computation, I started looking for used hardware, primarily on eBay. From January to March of 2019, I started purchasing equipment that would be required for the attempt: a new beefy server, many hard drives, various PCI-E cards, a tape libary and drive, to name just a few. Below is a picture of my home lab in it's entirety (minus an Asus ESC8000 G4 in my office). Everything inside the red box is used for the Pi computation.

<img src="/images/pi-record-attempt-setup.jpg" alt="record attempt hardware" style="height:800px;"/>

1. HP PROLIANT DL580 Gen8
    * This server (Ubuntu 18.10) performs the Pi computation. The final compressed Pi digits will be transferred to the Dell R720xd via a cifs-mounted directory (/drives/main_output).
    * (4) Intel Xeon E7-4880V2 2.5GHz 15C/30T CPU
    * 320GB DDR3 PC3-8500R ECC RAM
    * (5) 2.5" 100GB SATA MLC SSD (RAID 6)
    * (4) 6Gb LSI 9207-8e Dual Port SAS HBA
        * Connected to each HPE D3600 disk shelf via single SFF-8088 to SFF-8644 cable 2m
    * Intel 10-Gigabit X540-AT2 Dual Port Ethernet Adapter
        * Dedicated link to Dell R720xd for Veeam backups
    * Intel 1-Gigabit I340-T4 Quad Port Ethernet Adapter
        * Connection to the Internet
2. (4) HP StorageWorks D3600
    * These drives are used in lieu of server memory to provide y-cruncher the resources required to calculate 50 trillion digits of Pi (mounted on the DL580 as /drives/1 - /drives/48). I originally used HP StorageWorks D2600 disk shelves, but I didn't realize they were limited the SATA backplane to 3Gb/s. I upgraded to the D3600 disk shelf which supports 6Gb/s SATA in June of 2019.
    * (12) 3.5" 6TB SATA 7.2K HGST Ultrastar He8 HDD (JBOD)
3. Dell PowerEdge R720xd
    * This server (Windows Server 2016) serves two primary purposes. First, it is connected to a HP tape library. A Veeam job is manually run each month, which backs up data from the 48 drives running the Pi computation directly to tape. Second, it has an additional RAID6 storage volume which will contain the final compressed Pi digits that y-cruncher generates at the end of the computation.
    * (2) Intel Xeon E5-2670 8C/16T CPU
    * 128GB DDR3 ECC RAM
    * (2) 2.5" 600GB SAS 10K Seagate HDD (RAID 1)
    * Emulex LPE 12002, Dual Port 8Gb Fiber Channel HBA
        * Connected to tape library using (2) LC-LC OM3 2m fiber cables
    * Dell PERC H810 Adapter 6Gb RAID Controller Card
        * Connected to Dell MD1200 disk shelf
4. Dell PowerVault MD1200
    * 40TB 7.2K SAS 3.5" (RAID6)
        * (7) HGST Hitachi 4TB
        * (5) Seagate Constellation ES.3 4TB
5. HPE StoreEver MSL4048 Tape Library
    * LTO-5 FC Tape Drive
    * (21) LTO Ultrium 5 Tape Data Cartridge 1.5TB

One of the first problems I had to address was backing up the ephemeral computational data (48 hard drives). A small subset of the total storage is required to be backed up (around 15TB). y-cruncher generates checkpoint files every so often, and I wrote a Linux bash script to parse the checkpoint file and remove any leftover/unncessary files from being backed up. After that, I have an ansible script which connects to the Windows server running on the Dell R720xd, and it modifies and executes a Veeam direct file-to-tape backup job automatically. The script is also able to provide an estimated completion time, since Veeam doesn't provide a time estimate natively (it's accurate enough for me at least). I will make these scripts publicly available in the near future.

Here is a rough timeline of the world record attempt (still ongoing):
  * Feb 12, 2019    - This is the day I first reached out to Alex Yee about his y-cruncher program. I told him I was interested in breaking the world record (at that time it was 22.4 trillion digits set by Peter Trueb[^4]). My initial thoughts were to use a NetApp DS4486, NetApp FAS3220 filer, and Dell R910 to compute the record. At this point I wasn't sure how many digits I was going to calculate, but I knew it was going to be at least 22.4 trillion in order to beat the record.
  * Feb 12, 2019    - Alex responds with answers to some questions that I had relating to accessing storage (I originally planned on using iSCSI) and server CPU/memory. I was planning to access the required disk storage using iSCSI on the FAS3220, but Alex pointed out that this would be much too slow even over a 10 Gigabit link. Based on his analysis, I decided to directly connect the storage to the server using SAS cables (SFF-8088). Details were sketchy if I could directly attach the NetApp DS4486 to the main server, or if I would have to use a NetApp filer. After conducting some research, I decided it would just be easier to use external JBOD (non-RAID) disk enclosures. At the time I was looking at using several Dell PowerVault MD1200 enclosures.
  * Feb 13, 2019    - I start looking at the HP Proliant DL580 Gen8 instead of the Dell PowerEdge R910, due to the fact it has a newer processor and higher core count (15/30T vs 10/20T). After some analysis and research, I decide to shoot for 50 trillion digits of Pi. I originally planned on using RAID with the hard drives required during the calculation, but the sheer amount of storage required prevented this possibility. In order to mitigate the risk of a hard drive failure, I decided to utilize a HP MSL4048 tape library, which holds a maximum of 48 tapes. The unit I purchased off of eBay just happened to come with 21 LTO-5 tapes (not mentioned in the auction), so that was an added bonus! I got around to purchasing a single full-height LTO-5 tape drive so that I would be able to backup the hard drives to tape. Around the time I was looking for tape backup solutions, Veeam was restructuring their licensing, and the community edition now supported tape backups for up to five hosts. Based on this information, I decide to download Veeam Backup & Replication 9.5 Community Edition and put it on another server connected to the tape library over fiber. After coming to the decision to calculate 50 trillion digits, I determined I would need around 281TB of storage, and after looking on eBay I decided to purchase four HP D2600 external JBOD disk enclosures and forty-eight 6TB SATA drives to meet this need. I also saw that I would need 44TB for the final compressed Pi output. I picked up a Dell MD1200 external JBOD enclosure and twelve 4TB SAS drives that would serve this purpose. While the main 48 drives are not in a RAID volume, the twelve 4TB SAS drives are running in RAID6 to provide drive redundancy.
  * March 14, 2019  - I finally receive all of the hardware necessary for the computation to start. I contact Alex and he recommends I run several tests (I/O performance analysis, swap multiply). I learn that Google has beaten the current world record and was suprised to find that they were interested in this type of computation. Nevertheless, I was still aiming for 50 trillion digits, 18.6 trillion more then they were able to achieve. I also saw that it cost them around $200,000[^5], which seems very expensive for this type of computation. At this point I have still spent less than $10,000.
  * Apr 1, 2019    - All of the testing is completed and the I/O seek values are adjusted to 12Mb/s based on the results (due to drives being limited to 3Gb/s on HP D2600). I start the world record computation.
  * April 2, 2019  - y-cruncher Status update: Series: S ( 17 ) 1.419%
  * Apr 3, 2019    - I log into the server and I am greeted with the following error message: "A Raid-File operation has failed." After reaching out and troubleshooting, Alex recommends adjusting the ulimit value on Linux (by default, Linux only allows you to have 1023 open files handles by default)[^6]. I set the ulimit value to a very high value in order to prevent this issue from happening again, and this fixes the issue.
  * April 4, 2019  - y-cruncher Status update: Series: S ( 16 ) 1.794%
  * April 4, 2019  - y-cruncher Status update: Series: S ( 15 ) 2.268%
  * April 5, 2019  - y-cruncher Status update: Series: S ( 14 ) 2.867%
  * April 6, 2019  - y-cruncher Status update: Series: S ( 13 ) 3.625%
  * April 8, 2019  - y-cruncher Status update: Series: S ( 12 ) 4.584%
  * April 10, 2019 - y-cruncher Status update: Series: S ( 11 ) 5.796%
  * April 13, 2019 - y-cruncher Status update: Series: S ( 10 ) 7.330%
  * April 16, 2019 - y-cruncher Status update: Series: S ( 9 ) 9.271%
  * April 21, 2019 - y-cruncher Status update: Series: S ( 8 ) 11.727%
  * April 27, 2019 - y-cruncher Status update: Series: S ( 7 ) 14.836%
  * May 6, 2019    - y-cruncher Status update: Series: S ( 6 ) 18.773%
  * May 16, 2019   - y-cruncher Status update: Series: S ( 5 ) 23.761%
  * May 20, 2019   - y-cruncher Status update: Series: S ( 5 ) 23.761%
  * May 31, 2019   - y-cruncher Status update: Series: S ( 4 ) 30.086%
  * June 15, 2019  - I email Alex asking if there would be any issue upgrading the 3Gb/s SATA D2600 external disk enclosures to the D3600 6Gb/s SATA enclosure. He says there won't be any issue as long as the drives and mount points are the same.
  * June 19, 2019  - y-cruncher Status update: Series: S ( 3 ) 38.116%
  * June 23, 2019  - I swapped out the D2600 disk enclosures with the D3600 model. I transfer all forty-eight drives to the new enclosures. I had to purchase new drive caddies as well (HP Gen8 model vs Gen5), since the new enclosure drive bays are less wide than the older model. I let Alex know the disk enclosure transfer appears to have been successful and based on the greatly improved transfer speed (it should speed up the computation finish by a month or two) -- write speed: 3.15 GiB/s vs 5.69 GiB/s, read speed: 3.08 GiB/s vs 5.90 GiB/s.

[^1]: <https://boinc.berkeley.edu/>{:target="_blank"}
[^2]: <https://cloud.google.com/blog/products/compute/calculating-31-4-trillion-digits-of-archimedes-constant-on-google-cloud>{:target="_blank"}
[^3]: <http://www.numberworld.org/y-cruncher/>{:target="_blank"}
[^4]: <http://www.numberworld.org/y-cruncher/news/2016.html#2016_11_15>{:target="_blank"}
[^5]: <https://youtu.be/BwkpNd2ceBk?t=459>{:target="_blank"}
[^6]: <https://easyengine.io/tutorials/linux/increase-open-files-limit/>{:target="_blank"}