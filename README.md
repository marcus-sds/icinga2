https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/getting-started
https://github.com/Icinga/icingaweb2/blob/master/doc/02-Installation.md

### icinga2
yum install https://packages.icinga.org/epel/6/release/noarch/icinga-rpm-release-6-1.el6.noarch.rpm<br>
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm<br>

yum install icinga2 -y<br>
yum install httpd mysql-server mysql -y<br>
yum install -y icinga2-ido-mysql<br>
yum install -y icinga-web<br>
yum install -y openssl-devel <br>
yum install -y icinga2-ido-mysql icinga-idoutils-libdbi-mysql<br>
yum install -y gcc # required for later plugin compilation <br>
yum install -y mailx # required for sending notifications<br>
yum install -y php-mysql   #could be already there due to depencies.<br>
yum install -y nagios-plugins-all<br>

chkconfig icinga2<br>
chkconfig httpd on<br>
chkconfig mysqld on<br>

service mysqld start<br>
service httpd start<br>
service icinga2 start<br>

mysql_secure_installation<br>

mysql -u root -p<br>
CREATE DATABASE icinga;<br>
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'localhost' IDENTIFIED BY 'password';<br>
quit<br>
mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/mysql.sql<br>

iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT<br>
service iptables save<br>
set enforce 0<br>

## icigaweb2
yum install icingaweb2 icingacli<br>
http://127.0.0.1/icingaweb2/setup<br>
icingacli setup config directory --group icingaweb2;<br>
icingacli setup token create;<br>
The newly generated setup token is: xxxxxxxx<br>

/etc/php.ini<br>
;date.timezone = UTC<br>
setenforce 0<br>

touch /var/www/html/index.html<br>

## icingaweb2 websetup
http://www.2daygeek.com/install-icinga-web2-on-centos-rhel-fedora-opensuse-ubuntu-debian-mint/5/<br>

## icinga command
icinga2 feature enable command

## icinga cli
/etc/init.d/icinga2 checkconfig<br>
/etc/init.d/icinga2 reload<br>
icinga2 object list --type Service --name *ping*<br>

## icinga api
icinga2 api setup<br>
/etc/init.d/icinga2 reload<br>

curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/hosts/ou-host' \
-d '{ "templates": [ "generic-host" ], "attrs": { "address": "[ip]", "check_command": "hostalive", "vars.os" : "Linux" } }' \
| python -m json.tool<br>

curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/hosts/ou-dns' \
-d '{ "templates": [ "generic-host" ], "attrs": { "address": "[ip]" } }' \
| python -m json.tool<br>

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/services/localhost!realtime-load' \<br>
-d '{ "templates": [ "generic-service" ], "attrs": { "check_command": "load", "check_interval": 1,"retry_interval": 1 } }'<br>
Example for a new CheckCommand object:<br>

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X PUT 'https://localhost:5665/v1/objects/checkcommands/mytest' \
-d '{ "templates": [ "plugin-check-command" ], "attrs": { "command": [ "/usr/local/sbin/check_http" ], "arguments": { "-I": "$mytest_iparam$" } } }'<br>

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X POST 'https://localhost:5665/v1/objects/hosts/example.localdomain' \<br>
-d '{ "attrs": { "address": "192.168.1.2", "vars.os" : "Windows" } }' \<br>
| python -m json.tool<br>
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

$ curl -k -s -u root:icinga -H 'Accept: application/json' -X DELETE 'https://localhost:5665/v1/objects/hosts/example.localdomain?cascade=1' | python -m json.tool<br>
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
yum install postgresql-server postgresql
chkconfig postgresql on
service postgresql start
RHEL/CentOS 7:
yum install postgresql-server postgresql
postgresql-setup initdb
systemctl enable postgresql
systemctl start postgresql
