---
title: "Calculating Pi: Hard Drive Backups"
excerpt_separator: "<!--more-->"
author: Timothy Mullican
permalink: calculating-pi-backups
comments: false
categories:
  - Pi
tags:
  - Pi
  - Math
  - World Record
  - Backups
date: 2019-06-27T00:45:08-05:00
last_modified_at: 2019-06-27T00:45:08-05:00
---
This is a continuation of my Pi world record attempt [series]({% post_url 2019-06-25-pi-world-record-attempt-details %}). This blog post will talk about how I handle backups during the world record attempt. Due to the vast amount of computation storage required, implementing RAID or other types of redundant storage were not feasible. To mitigate the risk of drive failure, I decided to purchase a tape library and drive. Tape is typically more stable than hard drives when stored under appropriate conditions, since there is no spinning media. One thing I was warned about is that y-cruncher will leave behind files on drives used during the computation. Not all of these files are required to be backed up, and the unneeded ones should be removed before starting a backup.

First, before starting a backup, I stop the y-cruncher program. This is to ensure the y-cruncher checkpoint file and files on disk are not overwritten. Next, since there wasn't an existing way to automatically remove the unneeded files from the hard drives, I wrote a simple Bash [script]({% post_url 2019-06-27-pi-world-record-checkpoint-remove-files %}) to delete files that are not required by the y-cruncher checkpoint file. After I have deleted the unneeded files from the drives, I have an ansible script which connects to the Dell R720xd running Windows Server 2016. The script executes a Veeam direct file-to-tape backup job on the fly. The script is also able to provide an estimated completion time, since Veeam doesn't provide a time estimate natively. I will make these scripts publicly available in the coming weeks. For my attempt (48 drives), the amount of data typically backed up each month is approximately 15TB, which takes around 1.5 days to write to multiple LTO-5 tapes in the tape library.