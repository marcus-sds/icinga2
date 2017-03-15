https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/getting-started
https://github.com/Icinga/icingaweb2/blob/master/doc/02-Installation.md

### icinga2
yum install https://packages.icinga.org/epel/6/release/noarch/icinga-rpm-release-6-1.el6.noarch.rpm
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm

yum install icinga2 -y
yum install httpd mysql-server mysql -y
yum install -y icinga2-ido-mysql
yum install -y icinga-web
yum install -y openssl-devel 
yum install -y icinga2-ido-mysql icinga-idoutils-libdbi-mysql
yum install -y gcc # required for later plugin compilation 
yum install -y mailx # required for sending notifications
yum install -y php-mysql   #could be already there due to depencies.
yum install -y nagios-plugins-all

chkconfig icinga2
chkconfig httpd on
chkconfig mysqld on

service mysqld start
service httpd start
service icinga2 start

mysql_secure_installation

mysql -u root -p
CREATE DATABASE icinga;
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'localhost' IDENTIFIED BY 'password';
quit
mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/mysql.sql

iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
service iptables save
set enforce 0

## icigaweb2
yum install icingaweb2 icingacli
http://127.0.0.1/icingaweb2/setup
icingacli setup config directory --group icingaweb2;
icingacli setup token create;
The newly generated setup token is: xxxxxxxx

/etc/php.ini
;date.timezone = UTC
setenforce 0

touch /var/www/html/index.html

## icingaweb2 websetup
http://www.2daygeek.com/install-icinga-web2-on-centos-rhel-fedora-opensuse-ubuntu-debian-mint/5/

## icinga command
icinga2 feature enable command

## icinga cli
/etc/init.d/icinga2 checkconfig
/etc/init.d/icinga2 reload
icinga2 object list --type Service --name *ping*

## icinga api
icinga2 api setup
/etc/init.d/icinga2 reload

curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/hosts/ou-host' \
-d '{ "templates": [ "generic-host" ], "attrs": { "address": "[ip]", "check_command": "hostalive", "vars.os" : "Linux" } }' \
| python -m json.tool

curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/hosts/ou-dns' \
-d '{ "templates": [ "generic-host" ], "attrs": { "address": "[ip]" } }' \
| python -m json.tool

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/services/localhost!realtime-load' \
-d '{ "templates": [ "generic-service" ], "attrs": { "check_command": "load", "check_interval": 1,"retry_interval": 1 } }'
Example for a new CheckCommand object:

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/checkcommands/mytest' \
-d '{ "templates": [ "plugin-check-command" ], "attrs": { "command": [ "/usr/local/sbin/check_http" ], "arguments": { "-I": "$mytest_iparam$" } } }'

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X POST 'https://localhost:5665/v1/objects/hosts/example.localdomain' \
-d '{ "attrs": { "address": "192.168.1.2", "vars.os" : "Windows" } }' \
| python -m json.tool
{
    "results": [
        {
            "code": 200.0,
            "name": "example.localdomain",
            "status": "Attributes updated.",
            "type": "Host"
        }
    ]
}

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X DELETE 'https://localhost:5665/v1/objects/hosts/example.localdomain?cascade=1' | python -m json.tool
{
    "results": [
        {
            "code": 200.0,
            "name": "example.localdomain",
            "status": "Object was deleted.",
            "type": "Host"
        }
    ]
}

# postgresql installation
RHEL/CentOS 5/6:
# yum install postgresql-server postgresql
# chkconfig postgresql on
# service postgresql start
RHEL/CentOS 7:
# yum install postgresql-server postgresql
# postgresql-setup initdb
# systemctl enable postgresql
# systemctl start postgresql
