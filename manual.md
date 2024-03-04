make sure debian/ubuntu is up to date
```bash
sudo apt-get update && sudo apt-get upgrade
```
install required packages:
```bash
sudo net-tools apt-get install apache2 libapache2-mod-perl2 mariadb-server libdatetime-perl libcrypt-eksblowfish-perl libcrypt-ssleay-perl libgd-graph-perl libapache-dbi-perl libsoap-lite-perl libarchive-zip-perl libgd-text-perl libnet-dns-perl libpdf-api2-perl libauthen-ntlm-perl libdbd-odbc-perl libjson-xs-perl libyaml-libyaml-perl libxml-libxml-perl libencode-hanextra-perl libxml-libxslt-perl libpdf-api2-simple-perl libmail-imapclient-perl libtemplate-perl libtext-csv-xs-perl libdbd-pg-perl libapache2-mod-perl2 libtemplate-perl libnet-dns-perl libnet-ldap-perl libio-socket-ssl-perl libmoo-perl libdbd-mysql-perl -y
```
if using ubuntu, remove the default "open-vm-tools" and replace it with "open-vm-tools-desktop"
```bash
sudo apt-get autoremove open-vm-tools
```
reboot your system
```bash
sudo apt-get install open-vm-tools-desktop
```
reboot your system
copy paste support into vmware workstation should work now
edit the mariadb.conf.d/50-otrs.cnf config file using nano, or your prefered editor (gedit is GUI based, hence easier to use, replace "nano" with gedit)
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-otrs.cnf
```
OR
```bash
sudo gedit /etc/mysql/mariadb.conf.d/50-otrs.cnf
```
paste the following code into the above mentioned file:
```bash
[mysqld]
max_allowed_packet=64M
query_cache_size=36M
innodb_log_file_size=256M
```
restart MariaDB
```bash
sudo service mariadb restart
```
create a new root user for mariaDB
```bash
sudo mysql -u root -p
```
now you enter the mMarinaDB shell automatically (if not, rerun the above command)
execute the following command in the MarinaDB shell, the shell will quit on it's own ("quit;")
```bash
CREATE DATABASE otrs DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci; 
GRANT ALL PRIVILEGES ON otrs.* TO otrs@localhost IDENTIFIED BY 'kamisama123';
quit;
```
enable the required apache2 modules (back in the main linux terminal)
```bash
sudo a2enmod perl
sudo a2enmod headers
sudo a2enmod deflate
sudo a2enmod filter
```
restart apache2
```bash
sudo service apache2 restart
```
create the OTRS user, and give the required permissions
```bash
sudo useradd -d /opt/otrs -c 'OTRS user' otrs && sudo usermod -aG www-data otrs
```
download otrs community edition (v6)
```bash
wget https://otrscommunityedition.com/download/otrs-community-edition-6.0.32.tar.bz2
```
extract and move it to the final directory
```bash
tar -jxvf otrs-community-edition-6.0.32.tar.bz2 && sudo mv otrs-community-edition-6.0.32 /opt/otrs
```
check if perl installed all the required libs
```bash
sudo perl /opt/otrs/bin/otrs.CheckModules.pl
```
create a new OTSR config file
```bash
sudo cp /opt/otrs/Kernel/Config.pm.dist /opt/otrs/Kernel/Config.pm
```
```bash
sudo nano /opt/otrs/Kernel/Config.pm
```
fill in the value DatabasePw='PWD' (where PWD is your password)
save the file using ctrl+O, ctrl+X
edit the OTRS config script:
```bash
sudo nano /opt/otrs/scripts/apache2-perl-startup.pl
```
remove the "#' before "use DBD::mysql ();  and "use Kernel::System::DB::mysql;"
so that:
```bash
#use DBD::mysql ();
#Kernel::System::DB::mysql;
```
becomes:
```bash
use DBD::mysql ();
Kernel::System::DB::mysql;
```
set the required folder permisions for the otrs directory:
```bash
sudo /opt/otrs/bin/otrs.SetPermissions.pl --web-group=www-data
```
enable apache2
```bash
sudo ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/sites-enabled/otrs.conf
```
check if every module is correctly loaded and installed:
```bash
sudo perl -cw /opt/otrs/bin/cgi-bin/index.pl 
sudo perl -cw /opt/otrs/bin/cgi-bin/customer.pl
sudo perl -cw /opt/otrs/bin/otrs.Console.pl
```
finally, restart apache2
```bash
sudo perl -cw /opt/otrs/bin/cgi-bin/index.pl
sudo perl -cw /opt/otrs/bin/cgi-bin/customer.pl
sudo perl -cw /opt/otrs/bin/otrs.Console.pl
```
(all 3 of hte above should return "syntax OK")
open the OTRS installer ON THE MACHINE:
```html5
http://localhost/otrs/installer.pl
```

