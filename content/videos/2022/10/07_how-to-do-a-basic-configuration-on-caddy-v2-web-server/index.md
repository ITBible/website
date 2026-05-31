---
title: "How to do a basic configuration on Caddy v2 Web Server"
description: "This is a basic walkthrough on configuring Caddy v2 Web Server."
summary: "This is a basic walkthrough on configuring Caddy v2 Web Server."
categories:
  - Linux
tags: [Tutorial, Caddy]
date: 2022-10-07
slug: how-to-do-a-basic-configuration-on-caddy-v2-web-server
authors:
    - "wizardtux"
---
{{< youtubeLite id="4bRHH6RToyU" label="How to do a basic configuration on Caddy v2 Web Server" >}}

Default Path:

```bash
sudo nano /etc/caddy/CaddyFile
```
Redirect with HTTP:

```text
tutorial.itbible.org:80 {
    redir https://google.com permanent
}
```
Redirect with HTTPS:
```text
tutorial.itbible.org:443 {
    redir https://google.com permanent
}
```
Reverse Proxy (Forces HTTPS with HTTP redirects to HTTPS):

```text
tutorial.itbible.org {
    reverse_proxy https://localhost:81
}
```
Basic HTML Web Page:
```text
tutorial.itbible.org {
    root * /var/www/basic/
    file_server
}
```
Dynamic PHP Web Page (with PHP8.1-FPM):
```text
tutorial.itbible.org {
    root * /var/www/php/
    file_server

    php_fastcgi unix//var/run/php/php8.1-fpm.sock

    try_files {path} /index.php?{query}
}
```