# pk-local
Test tool for conferring polkit console privileges on remote sessions

Please refer to the LICENSE file for the terms under which this software
is distributed, and under which it may be used.

# Description
When a user is physically present and logged on in front of a graphical
workstation, it is usual for modern systems to confer additional privileges
on that user. These privileges may enable the local user to do activities
like the following:-

- Reboot the system
- Shutdown the system
- Install updates
- Reconfigure hardware

Remote users generally don't get to do these things without additional
authentication.

Part of the system software on the workstation will be responsible for
determining whether a user is local or remote, and assigning privileges
accordingly. On many modern Linux distributions the subsystem responsible
for this is [Polkit](https://en.wikipedia.org/wiki/Polkit) [wikipedia.org]

Distributions using Polkit are responsible for deciding what privileged
actions the user might take, and how requests to perform these actions
will be handled for local users and for remote users.

Remote graphical logins are fairly rare in the Linux world. Also, Linux
systems are put to a variety of uses, from single-user workstations to
multi-user servers. This means that the default policies for
remote graphical sessions are rarely well tested, or even satisfactory.
The single user of a small Linux computer might be quite happy rebooting a
machine remotely; this is unlikely to be the case on a multi-user server.

This script provides a way for remote users who are members of a specific
UNIX group to obtain the privileges normally conferred on local users.

# FAQ
## Why should I use this script?
This script is useful if you suspect that Polkit configuration is
the reason why a remote graphical session is not behaving the way you
think it should.

On systems running versions of Polkit later than 0.105, you can easily
log the overrides that are being made by this script. This enables you to
craft Polkit rules which are tailored to you own use-case.

Using this script successfully needs some knowledge of Polkit. There are
plenty of resources on the Internet to help you with this. This script can
also help you learn about Polkit to some extent.

## Why should I not use this script?
This script is intended as a debugging aid. It is not intended for use
in a production environment, and certainly not on Internet connected systems.

Please do not run the script on anything other than an offline
system, unless you understand the security implications of what you are doing.

## How do I use the script?
Simply run the script as a privileged user, and pass in the `--enable` flag.
Here is an example from Ubuntu 22.04 LTS:-

```
$ sudo ./setup-pk-local --enable
- Generating /etc/polkit-1/localauthority/50-local.d/pk-local.pkla.w2s2KDKH6g...
- Renamed /etc/polkit-1/localauthority/50-local.d/pk-local.pkla.w2s2KDKH6g to /etc/polkit-1/localauthority/50-local.d/pk-local.pkla

  All users in group 'pk-local' now have console privileges when logged in
  from anywhere, over any connection.

   ***************************************************************************
   ** This is intended for testing purposes only.                           **
   **                                                                       **
   ** DO NOT expose systems set up in this manner to wider networks,        **
   ** particularly the Internet. Anyone obtaining access to your system via **
   ** a user in group 'pk-local' may be able to obtain complete control     **
   ** of your system, and the information it contains.                      **
   ***************************************************************************

  To reverse this operaton, re-run the script with the --disable switch

Current users in group pk-local : testuser
```

After doing this, you may need to create the pk-local group and add users to
it for testing. You should also restart the Polkit daemon so that your
changes are read in.

## Can I reverse the actions of this script?
Run the script as a privileged user and pass in the `--disable` flag:-

```
$ sudo ./setup-pk-local --disable
- Removing generated polkit overrides in /etc/polkit-1/localauthority/50-local.d/pk-local.pkla
```

Then restart the Polkit daemon.

## How do I get logging from the script?
This section applies to versions of Polkit after 0.105. Significantly this does
NOT include:-
- Ubuntu 22.10 and earlier
- Debian 11 and earlier

Run the script, and add a user in the pk-local group. On another terminal login (from `ssh` say), re-run polkitd with the --replace flag. The log in as the target user and run the operation for which you wish to get tracing.

Here is a trace for the above operations on Fedora 38, running the
`gnome-software` application:-

```
$ sudo /usr/lib/polkit-1/polkitd --replace
Successfully changed to user polkitd
15:55:30.027: Loading rules from directory /etc/polkit-1/rules.d
15:55:30.028: Loading rules from directory /usr/share/polkit-1/rules.d
15:55:30.049: Finished loading, compiling and executing 18 rules
Entering main event loop
Connected to the system bus
15:55:30.060: Acquired the name org.freedesktop.PolicyKit1 on the system bus
. . .
pk-local: action=[Action id='org.freedesktop.Flatpak.appstream-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.packagekit.system-network-proxy-configure'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.packagekit.trigger-offline-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.packagekit.system-sources-refresh'] user=testuser override=yes
15:56:08.834: Registered Authentication Agent for unix-process:17069:174212 (system bus name :1.305 [flatpak remotes --system --show-disabled --columns=name,filter,options], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_GB.UTF-8)
pk-local: action=[Action id='org.freedesktop.bolt.enroll'] user=testuser override=auth_admin_keep
pk-local: action=[Action id='org.freedesktop.packagekit.trigger-offline-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.NetworkManager.network-control'] user=testuser override=yes
15:56:08.966: Unregistered Authentication Agent for unix-process:17069:174212 (system bus name :1.305, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_GB.UTF-8)
pk-local: action=[Action id='org.freedesktop.packagekit.trigger-offline-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.packagekit.trigger-offline-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.bolt.enroll'] user=testuser override=auth_admin_keep
pk-local: action=[Action id='org.freedesktop.packagekit.trigger-offline-update'] user=testuser override=yes
pk-local: action=[Action id='org.freedesktop.NetworkManager.network-control'] user=testuser override=yes
```

From this log, the user can see the overrides being made by the script, and tailor a rules file specific to their own requirement.

## How do I write a rules file?
You'll need to find out about Polkit for this. This is not a Polkit tutorial, but there are plenty of resources online to help you with this

## I'm running Ubuntu 22.04 LTS. How do I get logging?
This operating system uses Polkit version 0.105. This Polkit version uses `.pkla` files instead of `.rules` files. This script will
detect version 0.105 and generate a `.pkla` files instead of a `.rules` file.

It is not possible to insert logging into these files. To see what Polkit is doing you will need to find another way to log what the daemon is doing.


