---
title: "Calculating Pi: Alerting on changes to y-cruncher Checkpoint File"
excerpt_separator: "<!--more-->"
author: Timothy Mullican
permalink: calculating-pi-alert-changes-checkpoint-file
comments: false
categories:
  - Pi
tags:
  - Pi
  - Math
  - World Record
  - Script
  - Text Message
date: 2019-07-13T23:04:52-05:00
last_modified_at: 2019-07-13T23:04:52-05:00
---
This is a continuation of my Pi world record attempt [series]({% post_url 2019-06-25-pi-world-record-attempt-details %}). Before I quit y-cruncher and start a backup, I need to know when the y-cruncher checkpoint file is modified. To accomplish this task, I use a program called incron[^1]. incron is able to detect file/directory changes and run commands as a result. In my case, I have a bash script that is executed when the y-cruncher Checkpoint File is modified, which will send me a text message (using Twilio API). Below is a copy of my incron configuration the script which send a text message. The script isn't perfect, but it works well enough for me.

```shell
$ incrontab -l
/home/tim/y-cruncher\ v0.7.7.9500-dynamic IN_CREATE,IN_DELETE /home/tim/send_text.sh $# $%
```

```bash
#!/bin/bash

# This script is executed by incron to send a text message (using Twilio) upon modification of the y-cruncher Checkpoint file. There is logic to only alert once for a single file change, so duplicate texts will not be sent.

#################################
# Copyright 2019 Timothy Mullican
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#################################

#!/bin/bash

account_sid="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
auth_token="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
state_file=~/files_already_alerted.txt

if [ -z "$1" ]; then
        output="New checkpoint file created."
else
        output="incron (y-cruncher): $@"
        filepath="/home/tim/y-cruncher v0.7.7.9500-dynamic/$1"
        if [[ -f "$filepath" ]]; then
                last_line=$(tail -n 1 "$filepath")
                # remove any special formatting or unprintable characters
                last_line=${last_line%$'\r'}
                output="$output (last line: $last_line)"
        fi
        [ ! -f $state_file ] && touch $state_file
        if grep "$1 $last_line" $state_file; then
                echo "Previously alerted on file. Exiting..."
                exit 0
        else
                echo "$1 $last_line" >> $state_file
        fi
fi

echo "$output"

from_number="+XXXXXXXXXX"
to_number="+XXXXXXXXXX"

echo "From: $from_number"
echo "To: $to_number"

curl -s -X POST -d "Body=$output" \
    -d "From=$from_number" -d "To=$to_number" \
    "https://api.twilio.com/2010-04-01/Accounts/$account_sid/Messages" \
    -u "$account_sid:$auth_token"
```

[^1]: https://www.howtoforge.com/tutorial/trigger-commands-on-file-or-directory-changes-with-incron/