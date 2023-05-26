# shared-app-daemon-scripts

This project contains scripts used for Enterprise Systems Software Engineering RHEL7/8 VMs.
They are installed commands to start and stop an app server instance,
and to deploy a new war file for the web app.

RHEL7: uses the systemd system instance (i.e., root's) to run Tomcat at the daemon user,
instead of via RHEL6's SysV init.  The prep and deploy scripts are now incorporated into
the app script, so they just call it, for backwards compatibility.

RHEL8: uses a systemd user instance (i.e., the daemon user's) to run Tomcat.
This is the preferred way, but RHEL7 does not support systemd --user.
Likewise, RHEL8's SELinux does not allow RHEL7's way, because the root systemd cannot
use an ExecStart nor PIDFile in the daemon user's home directory,
even when run as the daemon user.

PREREQUISITES

The following must be set up prior to installation:

1. The daemon user and group must be created on the host.
   * `CATALINA_BASE` tree in the daemon user's `$HOME/tomcat`
   * unique server shutdown port and proxied HTTPS connector port,
     domain proxyName, and host IP address in `$HOME/tomcat/conf/server.xml`
     and [Where is My Application](https://docs.google.com/spreadsheets/d/1cJUxgyLXftlbgYxM2DSE7X8Lof55miwoff1avm7DtRI/edit?pli=1#gid=0)
   * `$HOME/tomcat/conf/server.xml` with Host localhost with AccessLogValve, e.g.,:
     ```
     <Valve className="org.apache.catalina.valves.AccessLogValve"
     directory="logs"  prefix="access_log." suffix=".txt"
     pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %D &quot;%{X-Forwarded-For}i&quot;"/>
     ```
   * RHEL8 only:  the root user (or sudo)
       * `loginctl enable-linger {{ user }}`
       * edits `/etc/systemd/journald.conf` to add `Storage=persistent` 
         to the `[Journal]` section, and then `systemctl restart systemd-journald`
   * RHEL7 only:  the root user (or sudo)
       * sets up the daemon user with sudo access
         to run `/bin/systemctl {start|stop|show|status} tomcat_{{ user }}`,
         preferably with options `--no-pager --lines=[0-9][0-9] ` before `status`.
       * copies `tomcat_Xd.service` to `/etc/systemd/system/tomcat_{{ user }}.service`
         and edits it to change `{{ user }}` to the daemon username, e.g., `casl2d`.
       * `systemctl daemon-reload`
       * `systemctl enable tomcat_{{ user }}`
1. The app server instance must be created.
   * App URL in [Where is My Application](https://docs.google.com/spreadsheets/d/1cJUxgyLXftlbgYxM2DSE7X8Lof55miwoff1avm7DtRI/edit?pli=1#gid=0)
   * Apache HTTPS reverse proxy configured with app URL (including domain and `CTX_NAME`)
     to host and connector port.
1. Users that administer the web app must be in the daemon user's group.
1. Users in the daemon user's group must be set up with sudo access to switch user to the daemon user.
1. This distribution must be unpacked in `/usr/local/bin/appDaemon`

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
$ ls -l /usr/local/bin/appDaemon
total 198
-rwxrwxr-x   1 root     root        8445 Aug  5 11:23 app-common
-rw-r--r--   1 root     root        2047 Aug  5 10:50 app.parameters
-rwxrwxr-x   1 root     root        2812 Aug  5 11:23 deploy-common
-rw-r--r--   1 root     root         945 Mar  3 11:17 jmxremote.access
-rw-r--r--   1 root     root         430 Mar  3 11:17 jmxremote.password
-rwxrwxr-x   1 root     root        3182 Aug  5 11:23 prep-common
-rwxrwxr-x   1 root     root        1367 Aug  5 11:23 sapp-common
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
INSTALLATION

1. Switch to the daemon user.  (e.g., `osu casl2d` or `sudo su - casl2d`)
1. In the daemon user's home directory, create symbolic links to the shared app scripts.
    ```
    $ ln -s /usr/local/bin/appDaemon/app-common app
    $ ln -s /usr/local/bin/appDaemon/sapp-common sapp
    ```
   The `deploy-common` and `prep-common` scripts can also be linked in the same way,
   but they are just for backwards compatibility;
   the use of `app prep` and `app deploy`, or `app provision`, is recommended.
1. Copy the following files to the daemon user's home directory:
    ```
      app.parameters
      jmxremote.access
      jmxremote.password
    ```     
1. Edit the jmxremote.password file and set the password.
1. Edit the app.parameters file and set these:
    ```
     APP_GRP
     CTX_NAME
     JMX_PORT
    ```
     and any other settings as needed.
1. RHEL8 only (as the daemon user):
   1. `mkdir -p $HOME/.config/systemd/user/` and copy `tomcat.service` to there (no editing required).
   1. Edit `$HOME/.bash_profile` to add `export XDG_RUNTIME_DIR=/run/user/$(id -u)`
      and reevaluate it, e.g., `. .bash_profile`
      (so that `systemctl` can find the user's `systemd`).
   1. `systemctl --user daemon-reload` to read the `tomcat.service` file.
   1. `systemctl --user enable tomcat` to enable automatic startup on reboot.