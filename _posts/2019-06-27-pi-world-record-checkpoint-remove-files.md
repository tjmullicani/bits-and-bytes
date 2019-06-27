This is a continuation of my Pi world record attempt [series] <{% post_url 2019-06-25-pi-world-record-attempt-details %}>. Below is the script I use to remove unneeded files from drives when generating a y-cruncher backup. It's quick and dirty, but does the job. If you neglect to remove unneeded files in between y-cruncher runs, you risk the disk running out of space and your computation failing. Anyways, I run this script right before executing the Veeam backup to tape job.

```bash
#!/bin/bash

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