# Vagrant-OSAD

This is an installation of the [stackforge openstack deployment](https://github.com/stackforge/os-ansible-deployment) in a Vagrant managed virtual environment.


I require a flexible but easy to install, Openstack deployer where I can quickly switch between different releases and different settings, i.e. I may want a setting with Cinder or one with Swift or one with a specific ML2 plugin..and so on. I found the stackforge openstack ansible deployer to be the best to suit my needs. I like the fact that the stackforge ansible deployer uses the latest patches from each release and is actively maintained. Its a plus too that tempest testing is built into it as well. But haven't learnt how to use that yet.

There are others out there, but none provides me the the flexibility I need. Some are listed below
* [devstack-vagrant](https://github.com/openstack-dev/devstack-vagrant)
* [vagrant-ansible-openstack](https://github.com/dguerri/vagrant-ansible-openstack)

In this repo, I will keep my specific designs in different branches. So for example there will be:

* kilo_no_storage
* kilo_with_cinder
* juno_no_storage

and so on.  For the very first release, I have "kilo_no_storage" as my first branch.

I plan to keep this vagrant setup up to date as least til end of 2016. Not sure if I'll still be working with Openstack after that. So as the stackforge ansible project upgrades, I will upgrade this project in line with it too.

# Hardware Requirements
I build on an Laptop, with 8GB RAM and 250GB just fine.

# Usage

This setup currently uses the libvirt vagrant provider. I would love to have it also use the virtualbox provider, so if someone wants to help me out with that , I would gladly accept your pull request.

1. Install [Vagrant](https://www.vagrantup.com/downloads.html)
2. Install [libvirt](https://help.ubuntu.com/lts/serverguide/libvirt.html). Run libvirt as a non-privileged user (preferred)
3. Install [vagrant-libvirt](https://github.com/pradels/vagrant-libvirt)
4. Get the latest stable Ubuntu release. Currently its 14.04 and convert it into a Vagrant box for libvirt.
5. Change the Vagrantfile to use the correct vagrant box you installed
6. Adjust the CPU and Memory Requirements for the ``stackserver`` in the Vagrantfile. It is set by default to the minimum of 4G RAM and 2 CPUs.
7. The deploy server needs to be configured first cause the SSH key generated is then copied to all the haproxy, repo and openstack server VM.

```
$ vagrant up deployserver
```

Bring up the rest of the VMs in a non parallel fashion, at least for the libvirt
provider
```
$ vagrant up --no-parallel
```

8. When the initial provisioning is done SSH into the deploy server, verify you can ping all the other VMs and then start the openstack deployer
```
$ vagrant ssh deployserver
(deployserver)$ sudo su -
(deployserver)# cd /root/osad/playbooks
(deployserver)# ping 192.168.50.10
(deployserver)# ping 192.168.50.11
(deployserver)# ping 192.168.50.12
(deployserver)# openstack-ansible haproxy-install.yml
(deployserver)# openstack-ansible setup-everything.yml
.... Wait 3-5 Hours.....

```
9. Prisitine openstack environment. No images installed in Glance or network provisioned like in Devstack.
10. To connect to horizon from the host OS, use ``https://localhost:8081``. The admin password is ``password123``

# Vagrant Topology

```
                                                                                                                  Host OS
                  +--------------------+        +-----------------------+      +--------------------------+
+---------------+ |                    | +----+ |                       | +--+ |                          |  +------------+
                  |  Host Mgmt Bridge  |        | Container Mgmt Bridge |      |   Tenant Network Bridge  |
                  |                    |        |                       |      |                          |      Vagrant Env
                  +--------+-----+-+--++        +--+----+----+-----+----+      +--------------------------+
                           |     | |  |            |    |    |     |
                           |     | |  |            |    |    |     +------------------+
                           | +---------------------+    |    |                        |
                           | |   | |  |---------------------------------------------------+
                           | |   +-------------------------+ |                        |   |
                           | |     |                    |  | |                        |   |
                           | |     |        ++----------+  | |                        |   |
                           | |     |         |             | |                        |   |
                  +--------+-+--++ |         |      +------+-+------------+         +-+---+-----------+
                  |             |  |         |      |   Repo Host         |         | HA Proxy Host   |
                  | Deploy      |  |         |      |        +----------+ |         |                 |
                  | Host        |  |         |      |        |  Repo LXC| |         |                 |
                  |             |  |         |      |        +----------+ |         |                 |
                  +-------------+  |         |      +---------------------+         +-----------------+
                                   |         +--------|
                   +---------------+-------------------------------------------------------------------------------+
                   |                          +--------------------------+                    Linux Bridge ML2     |
                   |                          |  Container Mgmt Bridge on|                    ----------------     |
                   |                          |  Openstack Host          |                    Nova Compute         |
                   |                          +--------------------------+                    ---------------      |
                   |         +-------------------+             +-----------------+     +---------------+           |
                   |         | Cinder API LXC    |             |  Heat API LXC   |     |  nova api lxc |           |
                   |         +-------------------+             +-----------------+     +---------------+           |
                   |         +-----------------------+         |  Heat Engine LXC|     +-----------------+         |
                   |         | Cinder Scheduler LXC  |         +-----------------+     |  nova cert lxc  |         |
                   |         +-----------------------+         |  Horizon LXC    |    ++-----------------+---+     |
                   |         +-----------------------+         +-----------------+    |   nova conductor lxc |     |
                   |         | Galera LXC            |         |   Keystone LXC  |    +----------------------+     |
                   |         +-----------------------+         +-----------------+    |   nova scheduler     |     |
                   |        +------------------------+         |   MemCached LXC |    |                      |     |
                   |        |  Glance LXC            |         +-----------------+    +----------------------+     |
                   |        +------------------------+        +-------------------+   +-----------------------+    |
                   |         +----------------------+         | Rsyslog LXC       |   |  Utility LXC          |    |
                   |         | RabbitMQ LXC         |         +-------------------+   +-----------------------+    |
                   |         +----------------------+                                                              |
                   |                                                                                               |
                   |                                                                                               |
                   +-----------------------------------------------------------------------------------------------+

```

The topology consists of 3 bridges on the Host OS and 3 VMs.

### Topology Info from Libvirt

#### Server list
```
virsh # list
 Id    Name                           State
----------------------------------------------------
 9     osad_deployserver              running
 10    osad_reposerver                running
 11    osad_haproxy                   running
 14    osad_stackserver               running
```

#### bridge list
```
virsh # net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 container_mgmt       active     no            yes
 default              active     yes           yes
 gatewayrtr           active     no            yes
 host_mgmt            active     no            yes
 vagrant-libvirt      active     no            yes
```

#### virtual interface configuration
```
virsh # domiflist osad_deployserver
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet0      network    vagrant-libvirt virtio      52:54:00:a7:33:01
vnet1      network    host_mgmt  virtio      52:54:00:7c:fb:d8
vnet2      network    container_mgmt virtio      52:54:00:72:f9:c8

virsh # domiflist osad_reposerver
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet3      network    vagrant-libvirt virtio      52:54:00:ea:d6:ee
vnet4      network    host_mgmt  virtio      52:54:00:26:98:76
vnet5      network    container_mgmt virtio      52:54:00:ba:63:f0

virsh # domiflist osad_haproxy
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet6      network    vagrant-libvirt virtio      52:54:00:9d:0c:bf
vnet7      network    host_mgmt  virtio      52:54:00:ce:9e:7d
vnet8      network    container_mgmt virtio      52:54:00:8d:8e:d6

virsh # domiflist osad_stackserver
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet9      network    vagrant-libvirt virtio      52:54:00:3f:7f:36
vnet10     network    host_mgmt  virtio      52:54:00:d7:88:2a
vnet11     network    container_mgmt virtio      52:54:00:e4:41:7e
vnet12     network    for_tenants virtio      52:54:00:dc:05:1f
```

### Virtual Server Information

#### Deployment Server
This is a 256MB RAM VM running the latest stable Ubuntu release. The initial vagrant ansible provisioning script installs a apt-cacher and the latest stackforge ansible deployer playbook for the release specified in the Vagrantfile node definition for ``deployserver`` The other VMs and containers within those VMs have modified sources.list to use the apt-cacher. So I make sure not to destroy this deployment server unless I really need to. It cuts the install time. Not sure by much. Never before using caching and after using caching.

#### HA Proxy Server
According to the ansible openstack installer, this HA proxy can be used for demo and testing purposes. In this case, the proxy service is used by the installer to find various services. It is configured before the openstack server is run. When running the ansible-playbook command in the ``playbooks`` directory of the stackforge ansible deployer, it runs a dynamic inventory script found in ``playbooks/inventory`` that reads the ``/etc/openstack_deploy/openstack_user_config.yml`` config and generates all the config the HA proxy needs to configure the service. So it knows all the IPs of the containers before hand and what ports each openstack service that needs to be proxied is using.

#### Repo server
This is configured with all the openstack git python packages specific to the release specified in the installer. I made into a separate VM because the process of downloading all the packages takes a long time. So after my first install, can destroy the openstack server and bring it up say with swift services instead of cinder services or if I really messed up. Just destroy the openstack server and start again and the install will take 1/2 hr less.

#### OpenStack server

I put all openstack services into one server. The stackforge ansible deployer puts most services into LXC containers. Only service actually running on the openstack server is Nova and Neutron Linux bridge agent. The other services are all running in LXC containers on the server. (at least that what I can tell so far)

Seems to work so far. I havent' done an install with Cinder but I make requirements for this in the VM by adding an extra 40G drive. It shows up as ``/dev/vda``

## Contributing

1. Fork it.
2. Create your feature branch (`git checkout -b my-new-feature`).
3. Commit your changes (`git commit -am 'Add some feature'`).
4. Push to the branch (`git push origin my-new-feature`).
5. Create new Pull Request.

## License and Authors
Author:: Stanley Kamithi`

License:: Apache 2.0
