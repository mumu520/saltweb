
saltweb 运维管理平台

部署环境说明

cents6.2
django 1.6.2
salt-master 2014.1.0-1
Python 2.6.6

saltweb部署
wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-2.1.tar.gz
tar zxf setuptools-2.1.tar.gz
cd setuptools-2.1
python setup.py install
cd ..
wget --no-check-certificate https://pypi.python.org/packages/source/p/pip/pip-1.5.tar.gz
tar zxf pip-1.5.tar.gz
cd pip-1.5
python setup.py install
cd ..
yum -y install gcc gcc-c++ python-devel
yum install php-devel mysql mysql-server mysql-devel nginx php-mysql php php-fpm MySQL-python
yum install salt-master salt-minion salt
yum install rrdtool-devel python-rrdtool 
pip install django
pip install uwsgi 
pip install apscheduler paramiko
sed -i 's/\(.*\)HAVE_DECL_MPZ_POWM_SEC/#\1HAVE_DECL_MPZ_POWM_SEC/' /usr/lib64/python2.6/site-packages/Crypto/Util/number.py
sed -i 's/\(.*\)mpz_powm_sec/#\1mpz_powm_sec/' /usr/lib64/python2.6/site-packages/Crypto/Util/number.py
mkdir /etc/salt/master.d
cat << EOF > /etc/salt/master.d/group.conf
nodegroups:
    master: testgroups
EOF
echo "auto_accept: True" >>/etc/salt/master
grep "^default_include: master.d/\*\.conf" /etc/salt/master >/dev/null 2>&1|| echo "default_include: master.d/*.conf" >> /etc/salt/master


/etc/init.d/mysqld start
chkconfig mysqld on
mysqladmin -uroot password 123
mysql -uroot -p123 -e "create database saltweb default charset utf8"

./manage.py syncdb 
#注：第一次执行syncdb会提示创建超级管理员，该账号为saltweb的登录账号及后台超级管理员
./manage.py runserver 0.0.0.0:8001

sed -i 's/^user.*/user   root;/' /etc/nginx/nginx.conf
cat << EOF > /etc/nginx/conf.d/virtual.conf
server {
    listen          80;
    server_name    www.hhr.com;
    charset utf-8;
    index index.html index.htm index.php;
    access_log  /var/log/nginx/hhr.access.log;
    error_log  /var/log/nginx/hhr.error.log;
    location / {
        uwsgi_pass     127.0.0.1:8008;
        include        uwsgi_params;
    }
    location /static/ {
        alias  /root/saltweb/saltweb/static/;
        index  index.html index.htm;
    }
}
EOF

注意：如果saltweb在/root目录下，必须修改nginx的配置文件改成root用户，否则访问css报403错误
python /root/saltweb/saltweb/init.py
python /root/saltweb/saltweb/salt_service.py start
python /root/saltweb/saltweb/salt_service.py status

测试
修改本机host，添加"172.16.1.237 www.hhr.com",访问http://www.hhr.com/salt
