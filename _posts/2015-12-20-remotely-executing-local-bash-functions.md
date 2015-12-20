---
layout: post
title: Remotely executing local Bash functions
---

While writing a bash script to help automate one of many repeated tasks, I ran into a problem. My script was becoming unmanageable. A requirement of the script was to connect to a remote host and execute a series of commands. To begin with, I started writing this functionality in the simplest way possible.

```bash
#!/bin/bash

USER="user"
HOST="host"

ssh $USER@$HOST "
    command1
    command2
    command3
    ...
"
```
 For a real basic script, this method would work fine. But in my case, I had variables that needed evaluating as well as some command substitutions.

 ```bash
#!/bin/bash

USER="user"
HOST="host"
SERVICE="/path/to/dir"

ssh -t $USER@$HOST "
    sudo $SERVICE/bin/shutdown.sh > /dev/null 2>&1
    until [[ \$(pgrep -u root -f $SERVICE) == '' ]]; do
        echo 'Stopping service...'
        sleep 5
    done

    # execute commands while service is down

    echo 'Starting service...'
    sudo nohup $SERVICE/bin/startup.sh > /dev/null 2>&1
"
 ```

Gross. I now had to be mindful of using single or double quotes, some command substitutions needed to occur locally while others remotely, and most annoyingly my syntax highlighting was broke. There has to be a better way.

The [declare](http://www.tldp.org/LDP/abs/html/declareref.html) command has a useful `-f` option that will list all defined functions in the current script. It also allows specifying which functions should be listed. By passing this list to the remote host the functions get defined remotely making them available to be executed.

```bash
#!/bin/bash

USER="user"
HOST="host"
SERVICE="/path/to/dir"

task () {
    SERVICE=$1

    sudo $SERVICE/bin/shutdown.sh > /dev/null 2>&1
    until [[ $(pgrep -u root -f $SERVICE) == "" ]]; do
        echo "Stopping service..."
        sleep 5
    done

    # execute commands while service is down

    echo "Starting service..."
    sudo nohup $SERVICE/bin/startup.sh > /dev/null 2>&1
}

ssh -t $USER@$HOST "$(declare -f task); task $SERVICE"
```

Now I can sanely add functionality without the added complexities and can enjoy my syntax highlighting!
