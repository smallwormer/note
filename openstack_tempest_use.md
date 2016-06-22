###OpenStack tempest 测试使用

####安装Tempest


#####准备基本环境

```
yum install vim git bash-completion python-virtualenv  gcc-c++.x86_64

yum install  libffi-devel.x86_64 openssl-devel   为后面的安装tempest做准备
```


#####获取tempest代码

创建虚拟机环境，在虚拟机环境中进行安装tempest

```

git clone http://git.openstack.org/openstack/tempest
virtualenv ~/.venv           
source ~/.venv/bin/active    
pip install  tempest/   

在安装过程中可能会出现 setuptools 版本过低的问题 目前版本至少要17.0
pip install -U  setuptools>=17.1
```

##### 创建tempest测试目录

```

1. mkdir openstack && cd openstack && tempest init  既可以创建 tempest执行，目录在 openstack 目录中
2.  直接 tempest init openstack  会自动创建openstack 目录 并生成需要文件
 目录文件为：
 
 (.venv)[root@tempest openstack]# tree
.
├── etc
│   ├── accounts.yaml.sample         测试时使用的自动创建的测试用户等
│   ├── config-generator.tempest.conf
│   ├── javelin-resources.yaml.sample
│   ├── logging.conf.sample          日志文件
│   ├── tempest.conf                 tempest测试主配置文件
│   ├── tempest.conf.sample
│   └── whitelist.yaml
├── logs
└── tempest_lock

3 directories, 7 files
(.venv)[root@tempest openstack]#
```

#####如何进行tempest测试OpenStack环境

* 配置tempest主配置文件

vim etc/tempest.conf

```
[DEFAULT]
[alarming]
[auth]               #此处配置keystone的认证
admin_username = admin              
admin_project_name = admin
admin_password = admin
admin_domain_name = default
[baremetal]
[compute]
image_ref = 49358e7b-8cf3-4fa8-8797-df1f4951ad5c     #这里配置镜像ID
image_ref_alt = 56ae3982-11b3-4de0-9c99-6b7e1b05c1d0
flavor_ref = 1								          #这里配置flavor
flavor_ref_alt =  215f6de6-229a-4753-ba62-8428aeb07045
fixed_network_name= network_name                    #该处配置创建虚拟机用的网络名称
min_compute_nodes = 3
min_microversion = 2.10
max_microversion = latest
[compute-feature-enabled]
change_password = true		                       #是否测试feature 功能 更改nova vm 密码
resize = true								       #resize测试(有resize的所有测试，有些是openstack master版本中存在的功能)
vnc_console = true
allow_duplicate_networks = true
[dashboard]
[data-processing]
[data-processing-feature-enabled]
[database]
[debug]
[identity]
uri_v3 = http://172.16.50.224:5000/v3
auth_version = v3
[identity-feature-enabled]
api_v2 = false
[image]
[image-feature-enabled]
deactivate_image = true
[input-scenario]
[negative]
[network]
project_network_cidr = 10.0.0.0/24
project_network_mask_bits = 28
project_networks_reachable = false
public_network_id = 070817b3-1199-4bd6-85a2-b2381f253161
floating_network_name = pub_net
dns_servers = 114.114.114.114
[network-feature-enabled]
[object-storage]
[object-storage-feature-enabled]
[orchestration]
[oslo_concurrency]
[scenario]
[service_available]
neutron = true
swift = false
heat = false
[stress]
[telemetry]
[telemetry-feature-enabled]
[validation]
run_validation = true
connect_method = floating
auth_method = keypair
ip_version_for_ssh = 4
image_ssh_user = cirros
[volume]
region  = all-compute
[volume-feature-enabled]
backup=false                           #是否测试cinder-backup
```
进行测试

```
进行所有测试(官方)
$ ostestr --regex '(?!.*\[.*\bslow\b.*\])(^tempest\.(api|scenario))'

$ostestr --regex '^tempest\.api\.(compute|identity|image|network|telemetry|volume)' > result.txt     #针对不同的组件进行测试


#进行单个为功能或者单个的函数进行测试
$ostestr --pdb tempest.api.network.test_metering_extensions.MeteringTestJSON 
#代表测试在是 在tempest/api/network/test_metering_extensions.py 中的 class MeteringTestJSON
```



注意事项：

```
在测试过程中如果出现了失败的问题，一般来说有可能是因为在
1. 测试的 openstack 环境不支持该功能
2. 测试的openstack中该功能没有启用
3. 测试的openstack环境中该功能确实不可用(在openstack中进行验证)
4. 测试的openstack环境中功能可用但是测试失败，请查看最新的tempest 代码是否是在openstack master 版本中正常，但是在当前环境中原来正常后来进行了更改
5. 查看 文件的更改 (git)
git log -p tempest/api/network/test_metering_extensions.py 查看该更改
```
