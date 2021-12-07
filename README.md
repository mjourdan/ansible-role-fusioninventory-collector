Role Name
=========

When using fusioninventory, you may want to avoid opening connections from your hosts in DMZ to your internal server.
This role will deploy a collector to gather inventories from your hosts in DMZ.

Requirements
------------

You already have a fusioninventory-compatible server on your LAN.

This role will be applied onto a separate php enabled server in dmz, to collect the inventories send by the hosts located in this zone.


Role Variables
--------------

Please refer to defaults/main.yml which describes all variables used by this role.


Dependencies
------------

### Collector

You'll need a php-enabled web server. This role is provided with a configuration for nginx. We have been using the following roles :

- nginxinc.nginx
- geerlingguy.php

### Agents

We recommend using ipr-cnrs.fusioninventory[1] to deploy the agent. For you're hosts in DMZ, you'll need to replace the "server_url" with the "collector_url". And you're all set.

[1]: https://github.com/ipr-cnrs/fusioninventory


Example Playbook
----------------

This will create a user with an ssh key for file synchronization, schedule a cron job to automatically fetch the data from your collector, and inject it into the web app:

    - hosts: inventory-server-01
			vars:
				fusioninventory_collector_webserver_hostname: inventory-collector.example.net
				fusioninventory_collector_injector_run: yes
				fusioninventory_collector_server_username: demo
				fusioninventory_collector_server_password: demo
				fusioninventory_collector_server_hostname: glpi-server
				fusioninventory_collector_injector_cron_state: present
      roles:
         - { role: mjourdan.fusioninventory-collector }

This will install the collector on an nginx webserver in dmz, and authorize synchronization from the inventory server :

    - hosts: dmz-inventory-collector-01
			vars:
			  fusioninventory_collector_state: present
				fusioninventory_collector_webserver_hostname: inventory-collector.example.net
				fusioninventory_collector_webserver_port: "443 ssl"
				fusioninventory_collector_fastcgi_pass: /var/run/php-fpm-fusioninventory-collector.sock
				php_enable_php_fpm: true
				php_fpm_pool_user: nginx
				php_fpm_pool_group: nginx
				php_fpm_pools:
					- pool_name: fusioninventory-collector
						pool_template: www.conf.j2
						pool_listen: "{{ fusioninventory_collector_fastcgi_pass }}"
						pool_pm: dynamic
						pool_pm_max_children: 5
						pool_pm_start_servers: 2
						pool_pm_min_spare_servers: 1
						pool_pm_max_spare_servers: 3
						pool_pm_max_requests: 500
				php_webserver_daemon: nginx
     roles:
         - { role: nginxinc.nginx }
         - { role: geerlingguy.php }
         - { role: mjourdan.fusioninventory-collector }


License
-------

GPLv3


Author Information
------------------

Mathieu Jourdan <m.jourdan@rennesmetropole.fr>
