# debian-install
A quick stuff of my debian server config

# 0 Preliminary stuff

## 0.1 System update

```console
apt update
apt upgrade
apt dist-upgrade
apt autoremove
apt autoclean
```

# 1 SSH Setup

## 1.1 Add authorized keys

✏️ `/root/.ssh/authorized_keys`

If you need to generate a key, you can use PuTTyGen or the following command:

```console
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

## 1.2 ssh config

✏️ `/etc/ssh/sshd_config`

Configuration:

* `Port <Change to whatever>`
* `PermitRootLogin prohibit-password`
* `PubkeyAuthentication yes`
* `PasswordAuthentication no`
* `PermitEmptyPasswords no`
* `ChallengeResponseAuthentication no`
* `UsePAM no`
* `X11Forwarding no`
* `PrintMotd no`
* `UseDNS no`
* `AcceptEnv LANG LC_*`

**[📝 Exemple file](samples/etc/ssh/sshd_config.md)**

----------

⚙️ Restart ssh and reconnect:

```console
service ssh restart
```

# 2 General config

## 2.1 Tools

## 2.1.1 git

Install:

```console
apt install git
git --version
```

Setting:

```console
git config --global user.name "Your name"
git config --global user.email "your@email.com"
git config --global core.editor "vim"
```

**[💡 Documentation (git-scm.com)](https://git-scm.com/book/fr/v2/Personnalisation-de-Git-Configuration-de-Git)**

## 2.1.2 vim

```console
apt install vim
```

## 2.2 Rsync

## 2.3 Cron

# 3 Security

## 2.1 Iptables

## 2.2 Fail2ban

## 2.3 UFW

```console
apt install ufw
ufw allow OpenSSH
ufw enable
ufw status
```

## 2.4 Other


# 4 Webserver

## 4.1 Apache2

Apache 2.4 will operate PHP

**[💡 Documentation (httpd.apache.org)](https://httpd.apache.org/)**

### 4.1.1 Install

```console
apt install apache2
```

Allow it in UFW:

```console
ufw allow 'Apache'
ufw status
```

Check its status:

```console
systemctl status apache2
```

Ensure that the service will be started at boot:

```console
systemctl enable apache2
```

### 4.1.2 Configuration

✏️ `/etc/apache2/ports.conf`
```apache
# If you just change the port or add more ports here, you will likely also have to change the VirtualHost statement in /etc/apache2/sites-enabled/000-default.conf
Listen 8085

# <IfModule ssl_module>
# 	Listen 443
# </IfModule>

# <IfModule mod_gnutls.c>
# 	Listen 443
# </IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

✏️ `/etc/apache2/conf-available/charset.conf`
```apache
# Read the documentation before enabling AddDefaultCharset.
# In general, it is only a good idea if you know that all your files have this encoding. It will override any encoding given in the files in meta http-equiv or xml encoding tags.

AddDefaultCharset UTF-8

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

✏️ `/etc/apache2/conf-available/javascrip-common.conf`
```apache
Alias /javascript /usr/share/javascript/

<Directory "/usr/share/javascript/">
	Options FollowSymLinks MultiViews
</Directory>
```

✏️ `/etc/apache2/conf-available/security.conf`
* `ServerTokens Prod`
* `ServerSignature Off`
* `TraceEnable Off`

✏️ `/etc/apache2/conf-custom/wordpress.conf`
```apache
<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteBase /
	RewriteRule ^index\.php$ - [L]
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule . /index.php [L]
</IfModule>
```

**Enable configurations**

```console
a2enconf charset  javascrip-common  security
```

**Enable mods**
```console
a2enmod rewrite http2 mime ssl
```

### 4.1.3 VirtualHosts config

* **[📝 Exemple file: Vhost simple](samples/etc/apache2/vhost-simple.md)**
* **[📝 Exemple file: Vhost wordpress](samples/etc/apache2/vhost-wordpress.md)**


## 4.2 Nginx

Nginx will be used as a reverse-proxy for Apache and NodeJS. It will operate static files.

**[💡 Documentation (nginx.org)](https://nginx.org/en/docs/)**

### 4.2.1 Install

```console
apt install nginx
```

### 4.2.2 Configuration

✏️ `/etc/nginx/nginx.conf`
* **[📝 Exemple file](samples/etc/nginx/nginx.conf.md)**

✏️ `/etc/nginx/conf.d/cache.conf`
```nginx
add_header Cache-Control "public, max-age=31536000, immutable";
```

✏️ `/etc/nginx/conf.d/charset.conf`
```nginx
map $sent_http_content_type $charset {
    default '';
    ~^text/ utf-8;
    text/css utf-8;
    application/javascript utf-8;
    application/rss+xml utf-8;
    application/json utf-8;
    application/manifest+json utf-8;
    application/geo+json utf-8;
}

charset $charset;
charset_types *;
```

✏️ `/etc/nginx/conf.d/default.conf`
```nginx
upstream apachephp {
    server <SERVER_IP>:<PORT_APACHE>;
}

server {
    charset utf-8;
    source_charset utf-8;
    override_charset on;
    server_name localhost;
}
```

✏️ `/etc/nginx/conf.d/headers.conf`
```nginx
# add_header X-Frame-Options "SAMEORIGIN";
# add_header X-XSS-Protection "1;mode=block";
add_header X-Content-Type-Options nosniff;
add_header Cache-Control "public, immutable";
add_header Strict-Transport-Security "max-age=500; includeSubDomains; preload;";
add_header Referrer-Policy origin-when-cross-origin;
add_header Content-Security-Policy "default-src 'self'; connect-src 'self' http: https: *.github.com api.github.com *.youtube.com; img-src 'self' data: http: https: *.gravatar.com youtube.com www.youtube.com *.youtube.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' http: https: www.google-analytics.com *.googleapis.com *.googlesynddication.com *.doubleclick.net youtube.com www.youtube.com *.youtube.com; style-src 'self' 'unsafe-inline' http: https: *.googleapis.com youtube.com www.youtube.com *.youtube.com; font-src 'self' data: http: https: *.googleapis.com *.googleuservercontent.com youtube.com www.youtube.com; child-src http: https: youtube.com www.youtube.com; base-uri 'self'; frame-ancestors 'self'";
```

✏️ `/etc/nginx/conf.d/proxy.conf`
```nginx
proxy_redirect			off;
proxy_set_header		Host		$host;
proxy_set_header		X-Real-IP	$remote_addr;
proxy_set_header		X-Forwarded-For	$proxy_add_x_forwarded_for;

client_max_body_size		10m;
client_body_buffer_size		128k;
proxy_connect_timeout		90;
proxy_send_timeout		90;
proxy_read_timeout		90;
proxy_buffer_size		16k;
proxy_buffers			32	16k;
proxy_busy_buffers_size		64k;
```

✏️ `/etc/nginx/conf.d/webmanifest.conf`
```nginx
add_header X-Content-Type-Options nosniff;
add_header Cache-Control "max-age=31536000,immutable";
```

✏️ `/etc/nginx/snippets/cache.conf`
```nginx
add_header Cache-Control "public, no-transform";
```

✏️ `/etc/nginx/snippets/expires.conf`
```nginx
map $sent_http_content_type $expires {
	default off;
	text/html epoch;
	text/css max;
	application/javascript max;
	~image/ max;
}
```

✏️ `/etc/nginx/snippets/fastcgi=php.conf`
```nginx
# regex to split $uri to $fastcgi_script_name and $fastcgi_path
fastcgi_split_path_info ^(.+\.php)(/.+)$;

# Check that the PHP script exists before passing it
try_files $fastcgi_script_name =404;

# Bypass the fact that try_files resets $fastcgi_path_info
# see: http://trac.nginx.org/nginx/ticket/321
set $path_info $fastcgi_path_info;
fastcgi_param PATH_INFO $path_info;

fastcgi_index index.php;
include fastcgi.conf;
```

✏️ `/etc/nginx/snippets/favicon_error.conf`
```nginx
location = /favicon.ico {
    access_log off;
    log_not_found off;
}
location = /robots.txt {
    return 204;
    access_log off;
    log_not_found off;
}
```

✏️ `/etc/nginx/snippets/gzip-config.conf`
```nginx
types {
	application/x-font-ttf           ttf;
	font/opentype                    ott;
}


gzip on;
gzip_disable "msie6";
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_min_length 256;
gzip_buffers 16 8k;
gzip_http_version 1.1;
#gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

# Compress all output labeled with one of the following MIME-types.
gzip_types
application/atom+xml
application/javascript
application/json
application/ld+json
application/manifest+json
application/rss+xml
application/vnd.geo+json
application/vnd.ms-fontobject
application/x-font-ttf
application/x-web-app-manifest+json
application/xhtml+xml
application/xml
font/opentype
image/bmp
image/svg+xml
image/x-icon
text/cache-manifest
text/css
text/plain
text/vcard
text/vnd.rim.location.xloc
text/vtt
text/x-component
text/x-cross-domain-policy;
# text/html is always compressed by gzip module
# don't compress woff/woff2 as they're compressed already
```

### 4.2.3 VirtualHosts config

* **[📝 Exemple file: Vhost simple](samples/etc/nginx/vhost-simple.md)**

## 4.3 PHP

## 4.4 NodeJS

## 4.4.1 NPM

## 4.4.2 PM2

PM2 is a production process manager for Node.js applications with a built-in load balancer. It allows you to keep applications alive forever, to reload them without downtime and to facilitate common system admin tasks.

**[💡 Documentation (npmjs.com)](https://www.npmjs.com/package/pm2)**

# 5 Databases

## 5.1 MariaDB

## 5.2 MongoDB

# 6 Letsencrypt

# 7 Webhooks

# 8 Mail server

## Postfix

# 9 FTP

# 10 Services

## 10.1 Monosnap (for screenshots)

## 10.2 VPN