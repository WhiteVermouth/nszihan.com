---
title: "Debian SSH 重启会话卡死问题解决"
date: "2018-09-14 12:42:31"
---

> When you shutdown or reboot your system, systemd tries to stop all services as fast as it can. That involves bringing down the network and terminating all processes that are still alive -- usually in that order. So when systemd kills the forked SSH processes that are handling your SSH sessions, the network connection is already disabled and they have no way of closing the client connection gracefully.

> Your first thought might be to just kill all SSH processes as the first step during shutdown, and there are quite a few systemd service files out there that do just that.

> But there is of course a neater solution (how it's "supposed" to be done): systemd-logind.
> systemd-logind keeps track of active user sessions (local and SSH ones) and assigns all processes spawned within them to so-called "slices". That way, when the system is shut down, systemd can just SIGTERM everything inside the user slices (which includes the forked SSH process that's handing a particular session) and then continue shutting down services and the network.

> systemd-logind requires a PAM module to get notified of new user sessions and you'll need dbus to use loginctl to check its status, so install both of those:

    apt-get install libpam-systemd dbus

[source](https://serverfault.com/questions/706475/ssh-sessions-hang-on-shutdown-reboot)
