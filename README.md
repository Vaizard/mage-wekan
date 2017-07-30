# mage-wekan

Ansible role to install wekan (kanban board). Depends on 
https://github.com/vaizard/mage-snapd role. See the defaults for configuration options, 
also note that daily backups will be performed via a cron script installed into /etc/cron.d.

Example playbook:

```
---

- hosts: wekan
  vars:
      wekan_root_url: 'https://wekan.example.com'
      wekan_mail_url: 'smtp://user:pass@mailserver.examples.com:25/'
      wekan_mail_from: 'wekan-admin@example.com'
  roles:
     - role: mage-update
     - role: mage-wekan
```

Cheatsheet:

```
# Logs
journalctl -u snap.wekan.wekan
journalctl -u snap.wekan.mongodb

# Services
service snap.wekan.wekan <action>
service snap.wekan.mongodb <action>

# For more help see
wekan.help

# Impossible to refresh wekan to a newer version? See the last comment in
# https://forum.snapcraft.io/t/error-when-updating-snap-and-cleaning-old-revisions/786/14
```

Upgrade workaround (lxd container version - for a different setup just drop the `lxc exec wekan --`)

```bash
lxc snapshot wekan # make a snapshot in case things go wrong
lxc exec wekan -- snap info wekan # see what version we got and what we can have
lxc exec wekan -- snap changes wekan # see where the upgrade broke
    # ID   Status  Spawn                 Ready                 Summary
    # 26   Error   2017-07-25T01:37:21Z  2017-07-30T14:51:26Z  Auto-refresh snap "wekan"
lxc exec wekan -- snap watch 26 # and see why it broke (problem unmounting snap-wekan-17)
    # ...
    # 2017-07-30T16:47:37+02:00 ERROR cannot remove snap file "wekan", will retry in 3 mins: [stop snap-wekan-17.mount] failed with exit status 1: Job for snap-wekan-17.mount failed. See "systemctl status snap-wekan-17.mount" and "journalctl -xe" for details.
    # 2017-07-30T16:50:37+02:00 ERROR cannot remove snap file "wekan", will retry in 3 mins: umount: /snap/wekan/17: not mounted
lxc exec wekan -- snap abort 26 # abort the erred task
lxc exec wekan -- snap changes wekan # aborting spawns another task that does nothing, we have to kill this too
    # ID   Status  Spawn                 Ready                 Summary
    # 26   Error   2017-07-25T01:37:21Z  2017-07-30T14:51:26Z  Auto-refresh snap "wekan"
    # 27   Doing   2017-07-30T14:51:27Z  -  
lxc exec wekan -- snap abort 27
lxc exec wekan -- service snap.wekan.wekan stop # stop wekan service
lxc exec wekan -- service snap.wekan.mongodb stop  # stop mongo service
lxc exec wekan -- ls -ahl /var/lib/snapd/snaps/wekan_*  # check if the problematic wekan-17 exists
lxc exec wekan -- rm /var/lib/snapd/snaps/wekan_17.snap # delte it
lxc exec wekan -- reboot # reboot
lxc exec wekan -- service snap.wekan.wekan stop # stop the services for the upgrade
lxc exec wekan -- service snap.wekan.mongodb stop # stop the services for the upgrade
lxc exec wekan -- snap refresh wekan --stable # upgrade (should complete successfully now)
lxc exec wekan -- service snap.wekan.mongodb start # start the services
lxc exec wekan -- service snap.wekan.wekan start  # start the services, all's good.
```
