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
lxc snapshot wekan
lxc exec wekan -- snap info wekan
lxc exec wekan -- snap changes wekan
    # ID   Status  Spawn                 Ready                 Summary
    # 26   Error   2017-07-25T01:37:21Z  2017-07-30T14:51:26Z  Auto-refresh snap "wekan"
lxc exec wekan -- snap watch 26
    # ...
    # 2017-07-30T16:47:37+02:00 ERROR cannot remove snap file "wekan", will retry in 3 mins: [stop snap-wekan-17.mount] failed with exit status 1: Job for snap-wekan-17.mount failed. See "systemctl status snap-wekan-17.mount" and "journalctl -xe" for details.
    # 2017-07-30T16:50:37+02:00 ERROR cannot remove snap file "wekan", will retry in 3 mins: umount: /snap/wekan/17: not mounted
lxc exec wekan -- snap abort 26
lxc exec wekan -- snap changes wekan
    # ID   Status  Spawn                 Ready                 Summary
    # 26   Error   2017-07-25T01:37:21Z  2017-07-30T14:51:26Z  Auto-refresh snap "wekan"
    # 27   Doing   2017-07-30T14:51:27Z  -  
lxc exec wekan -- snap abort 27
lxc exec wekan -- service snap.wekan.wekan stop
lxc exec wekan -- service snap.wekan.mongodb stop
lxc exec wekan -- ls -ahl /var/lib/snapd/snaps/wekan_*
lxc exec wekan -- rm /var/lib/snapd/snaps/wekan_17.snap
lxc exec wekan -- reboot
lxc exec wekan -- service snap.wekan.wekan stop
lxc exec wekan -- service snap.wekan.mongodb stop
lxc exec wekan -- snap refresh wekan --stable
lxc exec wekan -- service snap.wekan.mongodb start
lxc exec wekan -- service snap.wekan.wekan start
```
