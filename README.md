# user-analysis.sh
A tool to keep linux users on the live system in parity with systemd sysuser.d defaults

## Why I wrote it
In systemd context, a "fully locked account" is just a user account that cannot be logged into interactively. This prevents direct login while allowing services to run under a specific user context for security isolation.
* No valid password (set to ! or *)
* Shell set to /usr/sbin/nologin or /bin/false
* Used only by systemd services via the User= directive

Systemd does not provide any official way to [convert existing accounts to fully locked system accounts](https://github.com/systemd/systemd/issues/37179). This is problematic as distros adjust their packages to adopt the [change-sysusers-to-fully-locked-system-accounts](https://archlinux.org/todo/change-sysusers-to-fully-locked-system-accounts). The package created system users generated in the past will not inherit new package defaults for the increased security of locked/expired status. 

These users need to be modified manually for this change... which is exactly why I wrote this script.

In addition to syncing user account that need fully locked status, user-analysis.sh also identifies orphaned users (those created by a package no longer on the system) and can automatically delete them.

## Sample usage

Running the script with no token outputs the usage:
```
% sudo ./bin/user-analysis.sh
Usage: ./bin/user-analysis.sh {query|fix}

query : compare user status from sysuser.d files vs the live system, identify
        any orphaned users, then report findings
fix   : lock any user that should be locked per sysfiles.d files & delete any
        orphaned users
```

## user-analysis.sh in action
Here is a rather old system where a number of users have been created years ago before fully locked was a thing.
```
% sudo ./bin/user-analysis.sh q
 >>> ⚠️  The following users should be regenerated (they have 'u!' directive but are unlocked):
     systemd-network rpcuser systemd-coredump named rpc uuidd systemd-oom
     nobodysystemd-journal-remote systemd-timesync systemd-resolve

 >>> ⚠️  Users that have been orphaned by non-present packages:
     colord git kodi lxdm nvidia-persistenced systemd-bus-proxy
     systemd-journal-gatewaysystemd-journal-upload usbmux
```

Let's fix that:
```
% sudo ./bin/user-analysis.sh f
userdel: group kodi not removed because it has other members.
 >>> Optionally remove homedirs which have been intentionally left on the file system
     /var/lib/colord /var/lib/kodi /var/lib/lxdm
```

And now:
```
% sudo ./bin/user-analysis.sh q
 >>> ✅ User accounts on the live system are in parity with sysuser.d settings

     🔒 Locked/expired     🔓 Unlocked/active
----------------------------------------------
     named                   alpm
     nobody                  avahi
     rpc                     bin
     rpcuser                 daemon
     systemd-coredump        dbus
     systemd-journal-remote  dhcpcd
     systemd-network         ftp
     systemd-oom             http
     systemd-resolve         mail
     systemd-timesync        polkitd
     uuidd                   rtkit
                             _talkd
                             tss

 >>> ✅ No orphaned users present

% sudo ./bin/user-analysis.sh f
 >>> ❌ No users need to be locked nor are there any orphaned users, nothing to do
```
