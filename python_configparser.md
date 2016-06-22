# python ConfigParser

#### 在程序中可以使用配置文件来灵活的使用一些参数来做方便程序的解析，配置文件的解析并不复杂，python 官方库中可以使用ConfigParser来做解析，python的ConfigParser解析的配置文件格式比较像INI的配置文件的格式，就是文件有多个[section]构成，每个section下有 key value 组成


####ConfigParser 使用说明

#### nova 使用的配置文件如下：

```
cat /etc/nova/nova.conf


[DEFAULT]
vif_plugging_timeout = 300
vif_plugging_is_fatal = True
use_neutron = True
[wsgi]
api_paste_config = /etc/nova/api-paste.ini

[database]
connection = mysql+pymysql://root:stackdb@127.0.0.1/nova?charset=utf8

[api_database]
connection = mysql+pymysql://root:stackdb@127.0.0.1/nova_api?charset=utf8

[privsep_osbrick]
helper_command = sudo nova-rootwrap $rootwrap_config privsep-helper --config-file /etc/nova/nova.conf
[keystone_authtoken]
memcached_servers = 172.16.40.240:11211
signing_dir = /var/cache/nova
```
#### 使用python ConfigParser 来读取文件

```
import ConfigParser,string,os,sys

cnf = ConfigParser.ConfigParser()
cnf.read('/etc/nova/nova.conf')

#######返回所有的section值列表
sections = cnf.sections()         # sections = ['DEFAULT', 'wsgi', 'database', 'api_database','privsep_osbrick', 'keystone_authtoken']
options =  cnf.options('database') #获得在database 下面的所有key options = ['connection']
connection = cnf.get('database', 'connection')  #得出 database 下配置的connectiond的值


#修改其中的一个值 在写回到配置文件
cnf.set('keystone_authtoken', 'signing_dir', '/var/cache/nova1')
cnf.write(open("/ect/nova/nova.conf", "w"))

#添加一个section,添加后也要写回去

cnf.add_section('test')
cnf.set('test', 'user', 'yyp')
cnf.write(open("/ect/nova/nova.conf", "w"))

# 删除一个section 或者options (只要进行了修改都要写回到文件)
cnf.remove_option('test','user')
cnf.remove_setion('test')
cnf.write(open("/ect/nova/nova.conf", "w"))
```



官方参考链接 https://docs.python.org/2/library/configparser.html
