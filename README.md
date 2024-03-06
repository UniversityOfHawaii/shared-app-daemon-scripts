# shared-app-daemon-scripts

This project contains scripts used for Enterprise Systems Software Engineering RHEL7/8 VMs.
They are installed commands to start and stop an app server instance,
and to deploy a new war file for the web app.

RHEL7: uses the systemd system instance (i.e., root's) to run Tomcat as the daemon user,
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
1. This distribution should be deployed in `/usr/local/bin/appDaemon`, for example, on RHEL8:
    ```
    $ ls -l /usr/local/bin/appDaemon
    total 60
    -rw-r--r--. 1 myiamd myiam   861 Jun 22 18:04 50-shared-app-daemon-scripts.conf
    -rwxr-xr-x. 1 myiamd myiam  7866 Jun 22 18:04 app-common
    -rw-r--r--. 1 myiamd myiam  2615 Jun 22 18:04 app.parameters
    -rwxr-xr-x. 1 myiamd myiam   120 Jun 22 18:04 deploy-common
    -rw-r--r--. 1 myiamd myiam   959 Jun 22 18:04 jmxremote.access
    -rw-r--r--. 1 myiamd myiam   462 Jun 22 18:04 jmxremote.password
    -rwxr-xr-x. 1 myiamd myiam   118 Jun 22 18:04 prep-common
    -rw-r--r--. 1 myiamd myiam  4977 Jun 22 18:04 README.md
    -rwxr-xr-x. 1 myiamd myiam 18174 Jun 22 18:04 sapp-common
    -rw-r--r--. 1 myiamd myiam   774 Jun 22 18:04 tomcat.service
    ```
1. The root user (or sudo) must:
    * on RHEL8:
       * `loginctl enable-linger {{ user }}`
       * install `50-shared-app-daemon-scripts.conf` as instructed in that file.
   * on RHEL7:
       * set up the daemon user with sudo access
         to run `/bin/systemctl {start|stop|show|status} tomcat_{{ user }}`,
         preferably with options `--no-pager --lines=[0-9][0-9] ` before `status`.
       * copy `tomcat_Xd.service` to `/etc/systemd/system/tomcat_{{ user }}.service`
         and edit it to change `{{ user }}` to the daemon username, e.g., `tapsd`,
         and `{{ group }}` to the daemon group, e.g., `taps`.
       * `systemctl daemon-reload`
       * `systemctl enable tomcat_{{ user }}`
1. The app server instance must be created.
   * App URL in [Where is My Application](https://docs.google.com/spreadsheets/d/1cJUxgyLXftlbgYxM2DSE7X8Lof55miwoff1avm7DtRI/edit?pli=1#gid=0)
   * Apache HTTPS reverse proxy configured with app URL (including domain and `CTX_NAME`)
     to host and connector port.
1. Users that administer the web app must be
   set up with sudo access to switch user to the daemon user,
   and added to the daemon user's group if necessary to access files.

INSTALLATION

1. Switch to the daemon user.  (e.g., `osu tapsd` or `sudo su - tapsd`)
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
1. Edit the `jmxremote.password` file and set the password.
1. Edit the `app.parameters` file and set these:
    ```
     CTX_NAME
     JMX_PORT
    ```
     and any other settings as needed.
   (`APP_USER` and `APP_GRP` are obsolete, and should be removed to avoid confusion.)
1. RHEL8 only (as the daemon user):
   1. `mkdir -p $HOME/.config/systemd/user/` and copy `tomcat.service` to there (no editing required).
   1. Edit `$HOME/.bash_profile` to add `export XDG_RUNTIME_DIR=/run/user/$(id -u)`
      and reevaluate it, e.g., `. .bash_profile`
      (so that `systemctl` can find the user's `systemd`).
   1. `systemctl --user daemon-reload` to read the `tomcat.service` file.
   1. `systemctl --user enable tomcat` to enable automatic startup on reboot.