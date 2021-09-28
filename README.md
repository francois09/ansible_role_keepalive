KeepAlive Ansible role
======================

Role to manage install and config of keepalive on a dual node env

Requirements
------------

Role Variables
--------------

```
keepalive__install: True                         # Should install or not
keepalive__configure: True 			 # Should configure or not
keepalive__vip: 192.168.0.254			 # The vip switched between instances
keepalive__interface: eth0			 # The network interface who support the VIP
keepalive__instance: default_instance		 # Name of the instance
keepalive__instance_state: MASTER		 # Initial state of instance
keepalive__instance_priority: 100		 # Instance priority
keepalive__virtual_router_id: 180		 # Router ID
keepalive__services:	      			 #
  - name: haproxy				 # Service name used to control MASTER/BACKUP state
    # interval: 2				 # How ofter check service (sec)
    # fall: 2					 # How much fail before setting to FAULT
    # rise: 2					 # How much success before setting to UP
    # weight: 2					 # FAULT transition delay
keepalive__authentication_password: 123456	 # VRRP password
keepalive__global_defs:		    		 #
  smtp_server: localhost 25			 #
  notification_email: [email1, email2 ]		 # 
keepalive__nopreempt: true     	      		 # If master should reclaim VIP when up again or not
keepalive__smtp_alert: true			 #
keepalive__notify_scripts:			 # Script launched when in STATE state (or all)
  STATE:
    username: root
    groupname: root
    shell: |
      #!/bin/bash
      echo "$(date) keepalived $@ " >> /var/log/keepalived.log
```

Dependencies
------------

None

Example Playbooks
-----------------

VIP stickiness
==============

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
==========
To send an e-mail when a transition state occur:

```yaml
keepalive__smtp_alert: true
keepalive__global_defs:
  smtp_server: localhost 25
  notification_email: [ 'foo@example.com]
```

Notification scripts
====================

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
