# Quick XMPP server deployment

## Prerequisites

- Ubuntu 20.04
- Prosody 0.12+
- Assigned domain name ***the.site***
- Assigned upload server domain name ***upload.the.site***
- Open ports 80, 443, 5222, 5269, 5280, 5281

Substitute ***{upload.}the.site*** everywhere with actual domains and ***email@email.com*** with the SSL certificate issuer's e-mail.

## Configure the system

```shell
sudo hostnamectl set-hostname the.site
sudo apt-get install certbot mercurial
```

## Configure the firewall

```shell
for p in 80 443 5222 5269 5280 5281; do sudo firewall-cmd --zone=public --permanent --add-port=$p/tcp; done
sudo firewall-cmd --reload
```

## Obtain Let's Encrypt SSL certificates

```shell
sudo certbot certonly -n --standalone --agree-tos --email email@email.com -d the.site -d upload.the.site
```

<https://serverspace.io/support/help/how-to-get-lets-encrypt-ssl-on-ubuntu>

## Install Prosody

```shell
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install prosody
```

<https://prosody.im/download/package_repository>

<https://prosody.im/download/start>

## Install 3rd party Prosody modules

```shell
cd /var/lib/prosody
sudo hg clone https://hg.prosody.im/prosody-modules prosody-modules
sudo ln -s -t modules ../prosody-modules/mod_http_upload/mod_http_upload.lua
```

<https://hg.prosody.im/prosody-modules>

## Configure Prosody

<details>

<summary>/etc/prosody/prosody.lua.cfg</summary>

```
log = "/var/log/prosody/prosody.log"
pidfile = "/var/run/prosody/prosody.pid"
plugin_paths = {"/var/lib/prosody/modules"}
https_certificate="certs/upload.the.site.crt" --< Substitute upload.the.site with actual upload server domain

modules_enabled = {
  "tls";
  "disco";
  "saslauth";
  "ping";
  "uptime";
  "lastactivity";
  "mam";
  "carbons";
  "http_files";
}

VirtualHost "the.site" --< Substitute the.site with actual domain
enabled = true
default_storage = "internal"
default_archive_policy = true
http_files_dir = "/var/www"
http_dir_listing = false
archive_expires_after = "1m"
authentication = "internal_hashed"
disco_items = {{"upload.the.site"}} --< Substitute upload.the.site with actual upload server domain
ssl = {
  key = "certs/the.site.key"; --< Substitute the.site with actual domain
  certificate = "certs/the.site.crt"; --< Substitute the.site with actual domain
}

-- Requires Prosody 0.12+
-- https://prosody.im/doc/modules/mod_http_file_share
Component "upload.the.site" "http_file_share" --< Substitute upload.the.site with actual upload server domain
http_file_share_expires_after = 30 * 24*60*60 -- 30 days file expiration period
http_file_share_size_limit = 10 * 1024*1024 - 10 mb file size limit
```

</details>

<https://prosody.im/doc/configure>

## Install the SSL certificates

```shell
sudo prosodyctl --root cert import /etc/letsencrypt/live
```

## Restart Prosody

```shell
sudo systemctl restart prosody
```

## Arrange for further certificate renewals

<details>

<summary>sudo crontab -e</summary>

```
@daily /usr/bin/certbot renew --deploy-hook "/usr/bin/prosodyctl --root cert import /etc/letsencrypt/live && /bin/systemctl restart prosody"
```

</details>
