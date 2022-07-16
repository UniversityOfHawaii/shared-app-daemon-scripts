# shared-app-daemon-scripts

This project contains scripts used for Enterprise Systems Software Engineering RHEL7 VMs. They are installed Commands to start and stop an app server instance and to deploy a new war file for the web app.

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
1. The app server instance must be created.
   * App URL in [Where is My Application](https://docs.google.com/spreadsheets/d/1cJUxgyLXftlbgYxM2DSE7X8Lof55miwoff1avm7DtRI/edit?pli=1#gid=0)
   * Apache HTTPS reverse proxy configured with app URL (including domain and `CTX_NAME`) to host and connector port.
   * Host root copies `tomcat_Xd.service` to `/etc/systemd/system/tomcat_{{ user }}.service`
   and edits it to change `{{ user }}` to the daemon username, e.g., casl2d.
1. Users that administer the web app must be in the daemon user's group.
1. Users in the daemon user's group must be set up with sudo access to switch user to the daemon user.
1. The daemon user must be set up with sudo access to run `/bin/systemctl {start|stop|status} tomcat_{{ user }}`.
1. This distribution must be unpacked in /usr/local/bin

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
$ ls -l /usr/local/bin/appDaemon
total 198
-rwxrwxr-x   1 root     root        8445 Aug  5 11:23 app-common
-rw-r--r--   1 root     root        2047 Aug  5 10:50 app.parameters
-rw-r--r--   1 root     root       78267 Mar  3 11:17 common-1.17-socketStatisticsAgent.jar
-rwxrwxr-x   1 root     root        2812 Aug  5 11:23 deploy-common
-rw-r--r--   1 root     root         945 Mar  3 11:17 jmxremote.access
-rw-r--r--   1 root     root         430 Mar  3 11:17 jmxremote.password
-rwxrwxr-x   1 root     root        3182 Aug  5 11:23 prep-common
-rwxrwxr-x   1 root     root        1367 Aug  5 11:23 sapp-common
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
INSTALLATION

1. Switch to the daemon user.  (e.g., osu casl2d)
1. In the daemon user's home directory, create symbolic links to the shared version.  For example, with bash:
    ```
    $ for i in app sapp deploy prep
    do
      ln -s /usr/local/bin/appDaemon/$i-common $i
    done
    ```
1. Copy the following files to the daemon user's home directory:
    ```
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
