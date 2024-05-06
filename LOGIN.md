# Login

> Warning: everything described here is a moving target and subject to change.

It is assumed that for each user ${USER}, there is a realm ${USER}, located at /ix/realm/${USER}.<br> 
It is also assumed that this realm contains all the programs necessary for work and logging into the system.<br> 

An example of what may be found in a user realm: https://github.com/stal-ix/ix/blob/main/pkgs/set/pg/ix.sh.

For all users allowed to login, the /bin/session script is set as the shell:

```
pg# cat /etc/passwd  | grep sess
pg:redacted:10000:10000:none:/home/pg:/bin/session
```

This script configures the user environment, some environment variables, creates temporary folders for the user, etc.:

https://github.com/stal-ix/ix/blob/main/pkgs/bin/session/scripts/session

This script also exports environment settings from the user realm:

```
. /ix/realm/${USER}/etc/env
```

Some programs may configure the necessary environment for their functionality when installed in the user realm:

https://github.com/pg83/ix/blob/main/pkgs/lib/gtk/4/env/ix.sh

(therefore, if you want to use a different login scheme, you need to ensure that this file is read during the login process)

Then, the script passes control to user-session, which is the configuration point, meaning you can replace this script with your own.

```
exec user-session
```

By default, `user-session` passes control to ${HOME}/.session, which is also a configuration point. Here you can start session programs like ssh-agent and launch the Wayland compositor you are using.
