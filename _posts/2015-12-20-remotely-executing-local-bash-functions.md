---
layout: post
title: Remotely executing local Bash functions
---

While writing a bash script to help automate one of many repeated tasks, I ran into a problem. My script was becoming unmanageable. A requirement of the script was to connect to a remote host and execute a series of commands. To begin with, I started writing this functionality in the simplest way possible.

```bash
#!/bin/bash

USER=$1
HOST=$2

ssh $USER@$HOST "
    command1
    command2
    command3
    ...
"
```
 For a real basic script, this method would work fine. But as requirements grew, this method became less favorable.

 ```bash
#!/bin/bash

USER=$1
HOST=$2
APP=$3
WEBAPP=$4

ssh $USER@$HOST "
    echo \"Stopping $APP tomcat service...\"
    sudo systemctl stop $APP

    echo \"Deploying app...\"
    cd $WEBAPP
    jar -xvf /tmp/${APP}.war

    echo \"Starting $APP tomcat service...\"
    sudo systemctl start $APP

    rm /tmp/${APP}.war
"
 ```

This isn't terrible... but it's not good either. String quoting was becoming a nuisance and I wanted proper syntax highlighting.

The [declare](http://www.tldp.org/LDP/abs/html/declareref.html) command has a useful `-f` option that will list all defined functions in the current script. It also allows specifying which functions should be listed. By passing this list to the remote host the functions get defined remotely making them available to be executed.

```bash
#!/bin/bash

USER=$1
HOST=$2
APP=$3
WEBAPP=$4

deploy () {
    APP=$1
    WEBAPP=$2

    echo "Stopping $APP tomcat service..."
    sudo systemctl stop $APP

    echo "Deploying app..."
    cd $WEBAPP
    jar -xvf /tmp/${APP}.war

    echo "Starting $APP tomcat service..."
    sudo systemctl start $APP

    rm /tmp/${APP}.war
}

ssh $USER@$HOST "$(declare -f deploy); deploy $APP $WEBAPP"
```

Now I can sanely add functionality without the added complexities and can enjoy my syntax highlighting!
