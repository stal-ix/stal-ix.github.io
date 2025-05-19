# Login

> Disclaimer: Everything described here is a moving target and is subject to change.

It is assumed that for each user ${USER} there is a realm ${USER} located at /ix/realm/${USER}.<br> 
It is also assumed that this realm contains all the programs needed to operate and log in to the system.<br> 

An example of what you might find in a user realm: [https://github.com/stal-ix/ix/blob/main/pkgs/set/pg/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/set/pg/ix.sh).

For all users who are allowed to log in, the /bin/session script is set as the shell:

```
pg# cat /etc/passwd  | grep sess
pg:redacted:10000:10000:none:/home/pg:/bin/session
```

This script sets up the user environment, some environment variables, creates temporary folders for the user, etc.:

[https://github.com/stal-ix/ix/blob/main/pkgs/bin/session/scripts/session](https://github.com/stal-ix/ix/blob/main/pkgs/bin/session/scripts/session)

The script then passes control to `user-session`, which is a configuration point, meaning you can replace this script with your own.

```
exec user-session
```

By default, `user-session` passes control to ${HOME}/.session, which is also a configuration point. This is where you can run session programs like ssh-agent and start the Wayland compositor you use.
