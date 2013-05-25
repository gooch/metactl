metactl
=======

**Status:** Nothing to see here yet.

Metactl is an experimental next-generation version of a service lifecycle management script I wrote for work.

Features I want to keep from the original:

* Friendly CLI
* Good Java/Jetty integration
* SCM-based deployments
* Both Solaris and Linux support
* Integration with OS service manager (SMF, Upstart, Systemd)

Features under consideration for metactl:

* Support for other programming languages
* Multiple service entry points (multiple daemons, crontab)
* Distributed deployments (eg bump all production instances of myapp to version 1.2.3)
* Dynamic registration with a load balancer
* Configuration inheritance (per-app, per-site, per-server)
* Optional per-app IP and DNS addresses for debugging and discovery
* Lightweight sandboxing (using containers/zones)

Configuration
-------------

For each application Metactl has three layers of configuration.

### Application defaults

A per-app config file is checked into version control as `.metactl`. It is used for 
configuration common across all instances of an application such as entry points
and universal defaults.

```ini
[env]
JOB_MANAGER_PORT = 9000

[daemon.webapp]
platform = jetty6
jvm.heap.max = 256m

[daemon.jobmanager]
platform = jvm
jvm.jar = jobmanager.jar %(JOB_MANAGER_PORT)
jvm.heap.max = 128m

[crontab]
daily housekeeping = 5 0 * * * curl http://localhost/housekeeping
harvest data on mondays = 10 0 * * 1 java -jar harvest.jar
```

### Site configuration

Site configuration overrides app defaults for a particular deployment environment
(devel, test, production etc). It lives in some central location that's yet to be
decided.

```ini
[env]
DB_URL = mysql://myapp:secret@mysql-prod.example.org/myapp

[build]
repo = svn://svn.example.org/myapp
tag = 1.2.3
```

### Node configuration

Node configuration overrides site configuration for a particular deployed instance
of an app on a particular server. It lives in `/etc/metactl/nodes/myapp.conf` on
the individual server.

```ini
[env]
# change port because 9000 is already taken on this server
JOB_MANAGER_PORT = 9001
```
