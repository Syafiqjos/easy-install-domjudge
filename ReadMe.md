# Easy Install DOMJudge VM
Quick and Easy install DOMJudge with 2 VM (Separated DOMServer and Judgehost)
DOMJudge : [DOMJudge](https://www.domjudge.org)
Download : [DOMJudge Download](https://www.domjudge.org/download)

## Install DOMServer VM
Reference : [DOMServer Manual](https://www.domjudge.org/docs/manual/7.3/install-domserver.html)

### Initialize
```
cd ~
```

### Download DOMJudge
```
wget https://www.domjudge.org/releases/domjudge-7.3.3.tar.gz
tar -xf domjudge-7.3.3.tar.gz
mv domjudge-7.3.3.tar.gz domjudge
```

### Install Requirements
```
sudo apt install acl zip unzip mariadb-server apache2 \
      php php-fpm php-gd php-cli php-intl php-mbstring php-mysql \
      php-curl php-json php-xml php-zip composer ntp

sudo apt install libcgroup-dev
sudo apt install libjsoncpp-dev
```

### Make DOMServer
```
cd ~/domjudge
./configure --prefix=$HOME/domjudge
make domserver
sudo make install-domserver
```

### Generate Database
```
cd ~/domjudge/domserver/bin
dj_setup_database genpass
dj_setup_database root -r install
```

### Setting up Apache (php 7.4)
```
ln -s ~/domjudge/domserver/etc/apache.conf /etc/apache2/conf-available/domjudge.conf
ln -s ~/domjudge/domserver/etc/domjudge-fpm.conf /etc/php/7.4/fpm/pool.d/domjudge.conf
a2enmod proxy_fcgi setenvif rewrite
a2enconf php7.4-fpm domjudge
service php7.4-fpm reload
service apache2 reload
```

## Install Judgehost VM
Reference : [Judgehost Manual](https://www.domjudge.org/docs/manual/7.3/install-judgehost.html)

### Initialize
```
cd ~
```

### Download DOMJudge
```
wget https://www.domjudge.org/releases/domjudge-7.3.3.tar.gz
tar -xf domjudge-7.3.3.tar.gz
mv domjudge-7.3.3.tar.gz domjudge
```

### Install Requirements
```
sudo apt install make sudo debootstrap libcgroup-dev lsof \
      php-cli php-curl php-json php-xml php-zip procps \
      gcc g++ default-jre-headless default-jdk-headless \
      ghc fp-compiler
```

### Make Judgehost
```
cd ~/domjudge
./configure --prefix=$HOME/domjudge
make judgehost
sudo make install-judgehost
```

### Add User
```
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-0
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-1
```

### Create chroot environment
```
cd judgehost/bin
./dj_make_chroot
```

### Edit grub
Change line of  ```/etc/default/grub```
To ```GRUB_CMDLINE_LINUX_DEFAULT="quiet cgroup_enable=memory swapaccount=1"```
Then ```update-grub```

### create cgroup
```
sudo systemctl enable create-cgroups --now
```

### Start judgedaemon
```
sudo systemctl enable domjudge-judgehost-0
sudo systemctl start  domjudge-judgehost-1
```