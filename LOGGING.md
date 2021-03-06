# Enable CloudWatch Logging

This optional step makes it easier to figure out problems.

If you wish to type along and have not already performed the instructions on the previous page,
which is [Create IAM Role for the Lambda and the Lambda Itself](REGISTER.md).

If you are resuming these instructions in a new shell, load the environment variables from `settings.py`:

```script
$ source settings.py
```

## CloudWatch Logging

See the [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-logging.html) for more information on enabling [CloudWatch logging](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
for a particular API Gateway stage.

### Tailing CloudWatch Logs From a Terminal
Use [cw](https://www.lucagrulla.com/cw/).

#### WSL & WSL2 Installation
To install on Windows Subsystem for Linux 1 & 2, which do not support `snap`:
```script
cd ~/Downloads
wget https://github.com/lucagrulla/cw/releases/download/v3.3.0/cw_amd64.deb && sudo dpkg -i cw_amd64.deb
cd -
```

### Installation on Other Debian-Based Linux Distros
To install on Debian/Ubuntu:

```script
snap install cw-sh
sudo snap connect cw-sh:dot-aws-config-credentials
sudo snap alias cw-sh.cw cw
```

### Usage
Tail all the streams in the CloudWatch Log group `log-group1` which was defined at the top of this section, showing times in the local time zone.
```script
$ cw tail log-group1 --local
```

If you are unsure what CloudWatch Log groups you have defined in the current AWS region, find all CloudWatch Log groups:
```script
$ cw ls groups
```
This should include `log-group1` which was defined at the top of this section.

Find all streams in a CloudWatch Log group for the current AWS region, for example `log-group1`:
```script
$ cw ls streams log-group1
```

Tail just the streams that are named with the prefix `log-stream-prefix` in the CloudWatch Log group `log-group1` which was defined for the current AWS region at the top of this section, showing times in the local time zone.
```script
$ cw tail log-group1:log-stream-prefix --local
```

#### Fuzzy Completion with `fzf`

[`fzf`](https://github.com/junegunn/fzf) works well with `cw` for tab completion.
Installation is documented [here](https://github.com/junegunn/fzf#using-linux-package-managers).
For example:

```script
$ cw tail -f "$(cw ls groups | fzf)"
```

Use <kbd>tab</kbd> to select multiple log groups:
```script
$ cw tail -f $(cw ls groups | fzf -m | tr '\n' ' ')
```

### Deleting CloudWatch Logs
```script
$ aws logs describe-log-groups --log-group-name-prefix log-group1 \
  | jq '.[][]["logGroupName"]' \
  cut -d '"' -f 2 | \
    while read LOG_GROUP
    do
      aws logs delete-log-group --log-group-name $LOG_GROUP
    done
```


## Next Step

Separate instructions are given for creating the simpler [HTTP API](HTTP_API.md) and the more flexible [REST API](REST_API.md).
