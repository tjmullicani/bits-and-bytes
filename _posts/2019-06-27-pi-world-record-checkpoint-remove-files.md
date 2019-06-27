---
title: "Calculating Pi: Remove unneeded y-cruncher files before a backup"
excerpt_separator: "<!--more-->"
author: Timothy Mullican
permalink: calculating-pi-remove-unneeded-files-before-backup
comments: false
categories:
  - Pi
tags:
  - Pi
  - Math
  - World Record
  - Script
date: 2019-06-27T00:45:08-05:00
last_modified_at: 2019-06-27T00:45:08-05:00
---
This is a continuation of my Pi world record attempt [series]({% post_url 2019-06-25-pi-world-record-attempt-details %}). Below is the script I use to remove unneeded files from drives when generating a y-cruncher backup. It's quick and dirty, but does the job. If you neglect to remove unneeded files in between y-cruncher runs, you risk the disk running out of space and your computation failing. Anyways, I run this script right before executing the Veeam backup to tape job.

```bash
#!/bin/bash

# This script analyzes a y-cruncher checkpoint file to remove any unneeded files in preparation for a backup to be run. y-cruncher MUST be stopped before running this in order to avoid potential data loss.

#################################
# Copyright 2019 Timothy Mullican
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#################################

checkpoint_file="$1"
declare -a files
files=()
num_drives=48

# From https://stackoverflow.com/a/14367368
array_contains2 () {
    local array="$1[@]"
    local seeking=$2
    local in=1
    for element in "${!array}"; do
        if [[ $element == $seeking ]]; then
            in=0
            break
        fi
    done
    return $in
}

if [ -z "$1" ]; then
        echo "y-cruncher Checkpoint file not specified!"
        exit 1
fi

if [ ! -f "$1" ]; then
        echo "File \`$1\' does not exist!"
        exit 1
fi

while read -r line; do
        f=$(echo "$line" | sed 's/Object\:[ \t]*//g')
        # remove any lingering non-printable characters
        f=${f%$'\r'}
        f=$(echo "$f" | sed $'s/[^[:print:]\t]//g')

        files+=($f)
done <<<$(grep Object "$checkpoint_file")

i=1
# loop over all the drives. In my case, the drives used during the computation started at /drives/1 and ended at /drives/48, but y-cruncher starts at zero, so we need to subtract the drive minus 1. An example: /drives/1/ycs-00-0 and /drives/9/ycs-08-0/.
while [ $i -le $num_drives ]; do
        # prepend 0 if drive 9 or less
        if [ $i -le 10 ]; then
                drive_path=/drives/$i/ycs-0$((i - 1))-0/*
        else
                drive_path=/drives/$i/ycs-$((i - 1))-0/*
        fi

        for file in $drive_path; do
                if array_contains2 files $(basename "$file"); then
                        continue
                else
                        echo "[drive#$i] Unneeded file: $file"
                        rm -rf "$file"
                fi
        done
        # increment counter
        i=$[$i+1]
done
```