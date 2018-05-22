# shared-app-daemon-scripts

This project contains scripts used for Enterprise Systems Software Engineering VMs. They are installed Commands to start and stop an app server instance and to deploy a new war file for the web app.

PREREQUISITES

The following must be set up prior to installation:

1. The daemon user and group must be created.
1. The app server instance must be created.
1. Users that administer the web app must be in the daemon user's group.
1. Users in the daemon user's group must be set up with sudo access to switch user to the daemon user.
1. This distribution must be unpacked in /usr/local/bin

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
$ ls -l /usr/local/bin/appDaemon
total 198
-rwxrwxr-x   1 root     root        8445 Aug  5 11:23 app-common
-rw-r--r--   1 root     root        2047 Aug  5 10:50 app.parameters
-rwxrwxr-x   1 root     root        1443 Aug  5 11:23 clean-app-common
-rw-r--r--   1 root     root       78267 Mar  3 11:17 common-1.17-socketStatisticsAgent.jar
-rwxrwxr-x   1 root     root        2812 Aug  5 11:23 deploy-common
-rw-r--r--   1 root     root         945 Mar  3 11:17 jmxremote.access
-rw-r--r--   1 root     root         430 Mar  3 11:17 jmxremote.password
-rwxrwxr-x   1 root     root        3182 Aug  5 11:23 prep-common
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
INSTALLATION

1. Switch to the daemon user.  (e.g., osu casl2d)
1. In the daemon user's home directory, create symbolic links to the shared version.  For example, with bash:
    ```
    $ for i in app clean-app deploy prep
    do
      ln -s /usr/local/bin/appDaemon/$i-common $i
    done
    ```
1. Copy the following files to the daemon user's home directory:
    ```
      common-1.17-socketStatisticsAgent.jar
      app.parameters
      jmxremote.access
      jmxremote.password
    ```     
1. Edit the jmxremote.password file and set the password.
1. Edit the app.parameters file and set these:
    ```
     APP_USER
     APP_GRP
     CTX_NAME
     JMX_PORT
    ```
     And any other settings as needed.
