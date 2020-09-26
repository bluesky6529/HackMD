# 申請Let's encrypt(使用Certbot)

## 安裝Certbot 

### CentOS (7.X)
```
yum install -y certbot
yum install -y python2-certbot-apache
```

### Ubuntu (18.04) 

```
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot
sudo apt-get install python-certbot-nginx
```


## Certbot 手動更新
手動(自備 HTTP 伺服器 、其他主機)
不需先指向目標服務器，直接使用DNS TXT認證即可。
> 可能失敗原因：DNS服務器並未認證好就過了
> 解決辦法：在challenge前先驗證，驗證方式如下 
> linux 可以使用dig去驗證 
> ```
> dig TXT _acme-challenge.domain_name
> ```
> windows 可以使用 nslookup 去驗證: 
> ```
> nslookup
> > set type=TXT
> > _acme-challenge.domain_name
> ```

command :
```
 certbot certonly --manual --config-dir /etc/letsencrypt --work-dir /etc/letsencrypt --logs-dir /etc/letsencrypt --preferred-challenges dns
 ```
 
 ## Certbot ACME Challenge
 先略
 
 ## Certbot 自動更新憑證
 三個月過期 設定cronjob 每天跑一次即可
 ```
 certbot renew
 ```
 
 ### Certbot 常用指令
 
1. 列出所有憑證及到期日
```
certbot certificates
```
2. 測試是否能更新
```
certbot renew --dry-run
```
3. 手動立即更新SSL憑證
```
certbot renew
```
4. 移除憑證
```
certbot delete
```
 
 ### Certbot 功能
```
root@certbot:~# certbot --help

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

  certbot [SUBCOMMAND] [options] [-d DOMAIN] [-d DOMAIN] ...

Certbot can obtain and install HTTPS/TLS/SSL certificates.  By default,
it will attempt to use a webserver both for obtaining and installing the
certificate. The most common SUBCOMMANDS and flags are:

obtain, install, and renew certificates:
    (default) run   Obtain & install a certificate in your current webserver
    certonly        Obtain or renew a certificate, but do not install it
    renew           Renew all previously obtained certificates that are near
expiry
    enhance         Add security enhancements to your existing configuration
   -d DOMAINS       Comma-separated list of domains to obtain a certificate for

  (the certbot apache plugin is not installed)
  --standalone      Run a standalone webserver for authentication
  --nginx           Use the Nginx plugin for authentication & installation
  --webroot         Place files in a server's webroot folder for authentication
  --manual          Obtain certificates interactively, or using shell script
hooks

   -n               Run non-interactively
  --test-cert       Obtain a test certificate from a staging server
  --dry-run         Test "renew" or "certonly" without saving any certificates
to disk

manage certificates:
    certificates    Display information about certificates you have from Certbot
    revoke          Revoke a certificate (supply --cert-path or --cert-name)
    delete          Delete a certificate

manage your account with Let's Encrypt:
    register        Create a Let's Encrypt ACME account
    update_account  Update a Let's Encrypt ACME account
  --agree-tos       Agree to the ACME server's Subscriber Agreement
   -m EMAIL         Email address for important account notifications

More detailed help:

  -h, --help [TOPIC]    print this message, or detailed help on a topic;
                        the available TOPICS are:

   all, automation, commands, paths, security, testing, or any of the
   subcommands or plugins (certonly, renew, install, register, nginx,
   apache, standalone, webroot, etc.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```