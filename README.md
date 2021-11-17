KeepAlive Ansible role
======================

Role to manage install and config of keepalive on a dual node env

Requirements
------------

Role Variables
--------------

```
keepalive__install: True                        # Should install or not
keepalive__configure: True 			# Should configure or not

keepalive__global_defs:		    		# Global definitions for Keepalive
  smtp_server: localhost 25
  notification_email: [email1, email2 ]

keepalive__notify_scripts:			# Notification scripts used in instances
  - name: logEvent
    username: root
    groupname: root
    shell: |
      #! /bin/bash
      echo "$(date) $@" >> /var/log/keepalived.log

keepalive__conf:				# Instances declarations
  - name: gateway				# Instance name
    vip: "{{ internal_gateway }}"
    interface: "{{ internal_gw_port }}"
    state: MASTER
    priority: 100
    router_id: 180
    password: "{{ keepalived__gateway_password }}"
    check:
      name: "ovs-vswitchd"
      script: "/usr/bin/killall -0 ovs-vswitchd"
      interval: 2
      fall: 2
      rise: 2
    notify:
      name: logEvent
      username: root
      groupname: root
    nopreempt: true
    smtp_alert: true
```

Dependencies
------------

None

Example Playbooks
-----------------

VIP stickiness
++++++++++++++

There is 2 servers in BACKUP mode and the `nopreempt` option. On start, server with
higher priority becomes MASTER and manages the VIP. If the MASTER fail, the other
server takes over the VIP and become MASTER. If the previous MASTER server comes back,
the VIP remains on the current server (thanks to `nopreempt` option) even with a lower
priority.

```yaml

# Node 1
keepalive__vip: 192.168.200.254
keepalive__authentication_pass: {{ password }}
keepalive__service:
  - name: haproxy
    script: "/usr/bin/killall -0 haproxy"
    interval: 2
    fall: 2
    rise: 2
    # no weight to transition to FAULT state
keepalive__instance: loadbalancer
keepalive__virtual_router_id: 60
keepalive__instance_state: BACKUP
keepalive__nopreempt: true
keepalive__instance_priority: 101

# Node 2
# (same config except: )
keepalive__instance_priority: 102
```

SMTP alert
++++++++++

To send an e-mail when a transition state occur:

```yaml
keepalive__smtp_alert: true
keepalive__global_defs:
  smtp_server: localhost 25
  notification_email: [ 'foo@example.com]
```

Notification scripts
++++++++++++++++++++

Execute a script for 'all' state transition, where valid keys are 'all', 'master', 'backup', 'fault'

```yaml
keepalive__notify_scripts:
  all:
    username: root
    groupname: root
    shell: |
        #! /bin/bash
        echo "$(date) keepalive $@" >> /var/log/keepalived.log
```

Execute a script when a server becomes MASTER:

```yaml
keepalive__notify_scripts:
  master:
    username: root
    groupname: root
    shell: |
        #! /bin/bash
        echo "$(date) keepalive I'am the MASTER" >> /var/log/keepalived.log
```

License
-------

GPL v3

Author Information
------------------

François TOURDE
