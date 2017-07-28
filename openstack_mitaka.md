---
title: openstack_mitaka 安装笔记
date: 2017-01-23 15:31:00
categories:
- 云计算
- openstack
tags:
- 云计算
- openstack
keywords: openstack, openstack_mitaka, centos7
---
> 此篇笔记上的内容均参考自openstack官方文档，我只是加以整理，本文记录了在CentOS7上搭建openstack基本环境的全过程

<!-- more -->

## 官方中文文档

**[openstack_mitaka for centos7](http://docs.openstack.org/mitaka/zh_CN/install-guide-rdo/)**

## 环境概述

- OS:           CentOS 7.3 1611
- controller:   管理IP:10.0.0.1/8     NAT:192.168.255.101/24      内存至少2G
- compute:      管理IP:10.0.0.2/8     NAT:192.168.255.102/24      内存至少2G

## 设置主机名

<pre><code class="language-bash line-numbers">## controller
hostnamectl set-hostname controller

## compute
hostnamectl set-hostname compute
</code></pre>

## 修改hosts

<pre><code class="language-bash line-numbers">## controller
--- /etc/hosts ---
...
10.0.0.1    controller
10.0.0.2    compute
--- /etc/hosts ---

scp /etc/hosts root@10.0.0.2:/etc/hosts
</code></pre>

## ntp时间同步

<pre><code class="language-bash line-numbers">## controller
yum -y install chrony

--- /etc/chrony.conf ---
...
allow 10.0.0.0/8
...
--- /etc/chrony.conf ---

systemctl enable chronyd
systemctl start chronyd

## compute
yum -y install chrony

--- /etc/chrony.conf ---
...
server controller iburst
allow 10.0.0.0/8
...
--- /etc/chrony.conf ---

systemctl enable chronyd
systemctl start chronyd


## 验证
chronyc sources -v
timedatectl
</code></pre>

## 安装openstack_mitaka

<pre><code class="language-bash line-numbers">## 所有节点(controller,compute)
yum -y install centos-release-openstack-mitaka
</code></pre>

## 基本组件

<pre><code class="language-bash line-numbers">## controller上操作 ##

## openstackclient
yum -y install python-openstackclient


## mariadb
yum install mariadb mariadb-server python2-PyMySQL mariadb-common mariadb-libs -y

--- /etc/my.cnf.d/mariadb-server.cnf ---
[mysqld]
...
collation-server = utf8_general_ci
character-set-server = utf8
...
--- /etc/my.cnf.d/mariadb-server.cnf ---

systemctl enable mariadb
systemctl start mariadb

mysql_secure_installation # 初始化


## mongodb
yum install mongodb-server mongodb -y

--- /etc/mongod.conf ---
bind_ip = 0.0.0.0
smallfiles = true
--- /etc/mongod.conf ---

systemctl enable mongod
systemctl start mongod


## rabbitmq-server
yum install rabbitmq-server -y

systemctl enable rabbitmq-server
systemctl start rabbitmq-server

rabbitmqctl add_user openstack 123456
rabbitmqctl set_permissions openstack ".*" ".*" ".*"


## memcached
yum install memcached python-memcached -y

systemctl enable memcached
systemctl start memcached
</code></pre>

## keystone认证服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 创建数据库
mysql -p123456

CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY '123456';
exit;


# 随机数
openssl rand -hex 10


# 安装组件
yum install openstack-keystone httpd mod_wsgi -y


# 配置
--- /etc/keystone/keystone.conf ---
[DEFAULT]
...
admin_token = 随机数

[database]
...
connection = mysql+pymysql://keystone:123456@controller/keystone

[token]
...
provider = fernet
--- /etc/keystone/keystone.conf ---

# 初始化
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone


# 配置httpd - wsgi
--- /etc/httpd/conf/httpd.conf ---
ServerName controller
--- /etc/httpd/conf/httpd.conf ---

--- /etc/httpd/conf.d/wsgi-keystone.conf ---
Listen 5000
Listen 35357

&lt;VirtualHost *:5000&gt;
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat &quot;%{cu}t %M&quot;
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    &lt;Directory /usr/bin&gt;
        Require all granted
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;

&lt;VirtualHost *:35357&gt;
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat &quot;%{cu}t %M&quot;
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    &lt;Directory /usr/bin&gt;
        Require all granted
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;
--- /etc/httpd/conf.d/wsgi-keystone.conf ---

systemctl enable httpd
systemctl start httpd


# 临时令牌
export OS_TOKEN=随机数
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3

openstack service create \
  --name keystone --description "OpenStack Identity" identity
  
openstack endpoint create --region RegionOne \
  identity public http://controller:5000/v3

openstack endpoint create --region RegionOne \
  identity internal http://controller:5000/v3

openstack endpoint create --region RegionOne \
  identity admin http://controller:35357/v3
  
openstack domain create --description "Default Domain" default

openstack project create --domain default \
  --description "Admin Project" admin
  
openstack user create --domain default \
  --password-prompt admin
  
openstack role create admin

openstack role add --project admin --user admin admin

openstack project create --domain default \
  --description "Service Project" service
  
openstack project create --domain default \
  --description "Demo Project" demo
  
openstack user create --domain default \
  --password-prompt demo
  
openstack role create user

openstack role add --project demo --user demo user


# 移除临时令牌
--- /etc/keystone/keystone-paste.ini ---
字段  [pipeline:public_api]   [pipeline:admin_api]    [pipeline:api_v3]
删除  "admin_token_auth"
--- /etc/keystone/keystone-paste.ini ---

unset OS_TOKEN OS_URL OS_IDENTITY_API_VERSION

# 创建用户凭证
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin token issue
  
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue
  
--- admin-openrc ---
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
--- admin-openrc ---

--- demo-openrc ---
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
--- demo-openrc ---

# 验证
. admin-openrc
openstack token issue
</code></pre>

## glance镜像服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 创建数据库
mysql -p123456

CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY '123456';
exit;

# 初始化
. admin-openrc

openstack user create --domain default --password-prompt glance
openstack role add --project service --user glance admin

openstack service create --name glance \
  --description "OpenStack Image" image
  
openstack endpoint create --region RegionOne \
  image public http://controller:9292
  
openstack endpoint create --region RegionOne \
  image internal http://controller:9292
  
openstack endpoint create --region RegionOne \
  image admin http://controller:9292
  
# 安装组件
yum install openstack-glance -y

# 配置
--- /etc/glance/glance-api.conf ---
[database]
...
connection = mysql+pymysql://glance:123456@controller/glance

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = 123456

[paste_deploy]
...
flavor = keystone

[glance_store]
...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
--- /etc/glance/glance-api.conf ---

--- /etc/glance/glance-registry.conf ---
[database]
...
connection = mysql+pymysql://glance:123456@controller/glance

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = 123456

[paste_deploy]
...
flavor = keystone
--- /etc/glance/glance-registry.conf ---

# 初始化
su -s /bin/sh -c "glance-manage db_sync" glance

# 启动服务
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service

# 验证
. admin-openrc
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
  
openstack image list
</code></pre>

## nova计算服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 创建数据库
mysql -p123456

CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY '123456';
exit;

# 初始化
. admin-openrc

openstack user create --domain default \
  --password-prompt nova
  
openstack role add --project service --user nova admin

openstack service create --name nova \
  --description "OpenStack Compute" compute
  
openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1/%\(tenant_id\)s
  
# 安装组件
yum install openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler -y
  
# 配置
--- /etc/nova/nova.conf ---
[DEFAULT]
...
enabled_apis = osapi_compute,metadata
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.0.1
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
...
connection = mysql+pymysql://nova:123456@controller/nova_api

[database]
...
connection = mysql+pymysql://nova:123456@controller/nova

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456

[vnc]
...
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
...
api_servers = http://controller:9292

[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
--- /etc/nova/nova.conf ---

# 初始化
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova

# 启动服务
systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service


## compute上操作 ##

# 安装组件
yum install openstack-nova-compute -y

# 配置
--- /etc/nova/nova.conf ---
[DEFAULT]
...
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.0.2
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456

[vnc]
...
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
...
api_servers = http://controller:9292

[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
--- /etc/nova/nova.conf ---

# 查看是否支持cpu虚拟化
egrep -c '(vmx|svm)' /proc/cpuinfo

# 启动服务
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service


## controller上操作 ##

# 验证
. admin-openrc
openstack compute service list
+----+--------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary             | Host       | Zone     | Status  | State | Updated At                 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth   | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  2 | nova-scheduler     | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  3 | nova-conductor     | controller | internal | enabled | up    | 2016-02-09T23:11:16.000000 |
|  4 | nova-compute       | compute1   | nova     | enabled | up    | 2016-02-09T23:11:20.000000 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
该输出应该显示三个服务组件在控制节点上启用，一个服务组件在计算节点上启用。
</code></pre>

## neutron网络服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 创建数据库
mysql -p123456

CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY '123456';
exit;

# 初始化
. admin-openrc

openstack user create --domain default --password-prompt neutron

openstack role add --project service --user neutron admin

openstack service create --name neutron \
  --description "OpenStack Networking" network
  
openstack endpoint create --region RegionOne \
  network public http://controller:9696
  
openstack endpoint create --region RegionOne \
  network internal http://controller:9696
  
openstack endpoint create --region RegionOne \
  network admin http://controller:9696
  
# 安装组件
yum install openstack-neutron openstack-neutron-ml2 \
  openstack-neutron-linuxbridge ebtables -y

# 配置
--- /etc/neutron/neutron.conf ---
[database]
...
connection = mysql+pymysql://neutron:123456@controller/neutron

[DEFAULT]
...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
rpc_backend = rabbit
auth_strategy = keystone
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123456

[nova]
...
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 123456

[oslo_concurrency]
...
lock_path = /var/lib/neutron/tmp
--- /etc/neutron/neutron.conf ---

--- /etc/neutron/plugins/ml2/ml2_conf.ini ---
[ml2]
...
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
...
flat_networks = provider

[ml2_type_vxlan]
...
vni_ranges = 1:1000

[securitygroup]
...
enable_ipset = True
--- /etc/neutron/plugins/ml2/ml2_conf.ini ---
  
--- /etc/neutron/plugins/ml2/linuxbridge_agent.ini ---
[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = True
local_ip = 10.0.0.2
l2_population = True

[securitygroup]
...
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
--- /etc/neutron/plugins/ml2/linuxbridge_agent.ini ---
  
--- /etc/neutron/l3_agent.ini ---
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
external_network_bridge =
--- /etc/neutron/l3_agent.ini ---

--- /etc/neutron/dhcp_agent.ini ---
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
--- etc/neutron/dhcp_agent.ini ---

--- /etc/neutron/metadata_agent.ini ---
[DEFAULT]
...
nova_metadata_ip = controller
metadata_proxy_shared_secret = 123456
--- /etc/neutron/metadata_agent.ini ---

--- /etc/nova/nova.conf ---
[neutron]
...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123456
service_metadata_proxy = True
metadata_proxy_shared_secret = 123456
--- /etc/nova/nova.conf ---

# 初始化
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

# 启动服务
systemctl restart openstack-nova-api.service
systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl enable neutron-l3-agent.service
systemctl start neutron-l3-agent.service


## compute上操作 ##

# 安装组件
yum install openstack-neutron-linuxbridge ebtables ipset -y

# 配置
--- /etc/neutron/neutron.conf ---
[DEFAULT]
...
rpc_backend = rabbit
auth_strategy = keystone

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123456

[oslo_concurrency]
...
lock_path = /var/lib/neutron/tmp
--- /etc/neutron/neutron.conf ---

--- /etc/neutron/plugins/ml2/linuxbridge_agent.ini ---
[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = True
local_ip = 10.0.0.2
l2_population = True

[securitygroup]
...
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
--- /etc/neutron/plugins/ml2/linuxbridge_agent.ini ---

--- /etc/nova/nova.conf ---
[neutron]
...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123456
--- /etc/nova/nova.conf ---

# 启动服务
systemctl restart openstack-nova-compute.service
systemctl enable neutron-linuxbridge-agent.service
systemctl start neutron-linuxbridge-agent.service


## controller上操作 ##

# 验证
. admin-openrc
neutron ext-list
neutron agent-list
+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
| 08905043-5010-4b87-bba5-aedb1956e27a | Linux bridge agent | compute1   | :-)   | True           | neutron-linuxbridge-agent |
| 27eee952-a748-467b-bf71-941e89846a92 | Linux bridge agent | controller | :-)   | True           | neutron-linuxbridge-agent |
| 830344ff-dc36-4956-84f4-067af667a0dc | L3 agent           | controller | :-)   | True           | neutron-l3-agent          |
| dd3644c9-1a3a-435a-9282-eb306b4b0391 | DHCP agent         | controller | :-)   | True           | neutron-dhcp-agent        |
| f49a4b81-afd6-4b3d-b923-66c8f0517099 | Metadata agent     | controller | :-)   | True           | neutron-metadata-agent    |
+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
输出结果应该包括控制节点上的四个代理和每个计算节点上的一个代理。
</code></pre>

## dashboard服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 安装组件
yum install openstack-dashboard -y

# 配置
--- /etc/openstack-dashboard/local_settings ---
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*']

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211'
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "Asia/Shanghai"
--- /etc/openstack-dashboard/local_settings ---

# 启动服务
systemctl restart httpd.service memcached.service

# 验证
http://controller/dashboard     域:default 用户:admin 密码:123456
</code></pre>

## cinder块存储服务

<pre><code class="language-bash line-numbers">## controller上操作 ##

# 创建数据库
mysql -p123456

CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY '123456';
exit;

# 初始化
. admin-openrc

openstack user create --domain default --password-prompt cinder
openstack role add --project service --user cinder admin

openstack service create --name cinder \
  --description "OpenStack Block Storage" volume
  
openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
  
openstack endpoint create --region RegionOne \
  volume public http://controller:8776/v1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  volume internal http://controller:8776/v1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  volume admin http://controller:8776/v1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
  
# 安装组件
yum install openstack-cinder -y

# 配置
--- /etc/cinder/cinder.conf ---
[database]
...
connection = mysql+pymysql://cinder:123456@controller/cinder

[DEFAULT]
...
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.0.1

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = 123456

[oslo_concurrency]
...
lock_path = /var/lib/cinder/tmp
--- /etc/cinder/cinder.conf ---

# 初始化
su -s /bin/sh -c "cinder-manage db sync" cinder

# 配置
--- /etc/nova/nova.conf ---
[cinder]
os_region_name = RegionOne
--- /etc/nova/nova.conf ---

# 启动服务
systemctl restart openstack-nova-api.service
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service


## compute上操作 ##

# 添加硬盘 /dev/sdb 50G

# lvm
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb

--- /etc/lvm/lvm.conf ---
devices {
...
filter = [ "a/sda/", "a/sdb/", "r/.*/"]

## 注意：
如果您的存储节点在操作系统磁盘上使用了 LVM，您还必需添加相关的设备到过滤器中。例如，如果 /dev/sda 设备包含操作系统：
filter = [ "a/sda/", "a/sdb/", "r/.*/"]

类似地，如果您的计算节点在操作系统磁盘上使用了 LVM，您也必需修改这些节点上 /etc/lvm/lvm.conf 文件中的过滤器，将操作系统磁盘包含到过滤器中。例如，如果``/dev/sda`` 设备包含操作系统：
filter = [ "a/sda/", "r/.*/"]
--- /etc/lvm/lvm.conf ---

# 安装组件
yum install openstack-cinder targetcli python-keystone -y

# 配置
--- /etc/cinder/cinder.conf ---
[database]
...
connection = mysql+pymysql://cinder:123456@controller/cinder

[DEFAULT]
...
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.0.2
enabled_backends = lvm
glance_api_servers = http://controller:9292

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 123456

[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = 123456

[lvm]
...
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm

[oslo_concurrency]
...
lock_path = /var/lib/cinder/tmp
--- /etc/cinder/cinder.conf ---

systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service

## controller上操作 ##

# 验证
. admin-openrc
cinder service-list
</code></pre>
