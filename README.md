# Installing LetsEncrypt Certificate in Amazon Linux 2

## Install and enable EPEL

```
cd /tmp

wget -O epel.rpm –nv https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

sudo yum install -y ./epel.rpm
```

## Install Certbot package

```
sudo yum install python2-certbot-apache.noarch
```
## Install LetsEncrypt certificate using Certbot package


Before running the certbot command, find out the document root of the website. By default, it will be /var/www/html. Also, make sure that both non-www domain name ( eg: amazonaws.com ) and www domain name ( eg : www.amazonaws.com ) points to Instance's IP address.
```
sudo certbot -d amazonaws.com -d www.amazonaws.com certonly --webroot -w /var/www/html
```
( Please make sure to replace amazonaws.com with your domain name )

The certificate will be installed to the location /etc/letsencrypt/live/{domainname}/fullchain.pem.
The private key will be installed to the location /etc/letsencrypt/live/{domainname}/privkey.pem.

## Editing SSL configuration file to include the newly installed certificate for the website.


Edit your SSL config file : 
```
sudo vim /etc/httpd/conf.d/ssl.conf
```
Set SSLCertificateFile to your Certificate path which is /etc/letsencrypt/live/{domainname}/fullchain.pem

Set SSLCertificateKeyFile to your Private Key path which is /etc/letsencrypt/live/{domainname}/privkey.pem

Restart httpd :
```
sudo service httpd restart
```

## Create cronjob for automatic LetsEncrypt cert renewal

```
sudo su
echo "30 2 * * Sun certbot renew" > /etc/cron.d/cerbot
```

## Add redirection rules in http config to redirect non-www domain and www domain to https.

Open /etc/httpd/conf/httpd.conf file
Find the line : <Directory "/var/www/html">
Add the following rules below that line
```
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.amazonaws.com [NC]
RewriteCond %{SERVER_PORT} 80
RewriteRule ^(.*)$ https://www.amazonaws.com/$1 [R,L]
RewriteCond %{HTTP_HOST} ^amazonaws.com [NC]
RewriteCond %{SERVER_PORT} 80
RewriteRule ^(.*)$ https://www.amazonaws.com/$1 [R,L]
```
Replace "amazonaws.com" with your domain name.

Restart apache after adding the rules :
```
sudo service httpd restart
```
