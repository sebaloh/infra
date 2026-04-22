# acme.sh

This document is a collection of important information from the original `acme.sh` [documentation](https://github.com/acmesh-official/acme.sh/wiki).

## Content

- [Installation](#installation)

## Installation

> ℹ️ **Information:** The following instructions use `Hetzner Cloud DNS` for validation and also configure `ntfy` as a notification service. See [DNS API](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) for providers and [notify hooks](https://github.com/acmesh-official/acme.sh/wiki/notify) for notification services.

First, for this installation, we need to be logged in as the root user.

```sh
sudo -i
```

After switching, create the necessary directories for storing SSL certificates and acme.sh configuration files.

```sh
# ToDo: Change domain name
mkdir -p /etc/nginx/certs/example.com
mkdir -p /var/lib/acme/dnsapi
mkdir -p /var/lib/acme/notify
mkdir -p /etc/acme/certs
```

Next, download and install `acme.sh` along with the `Hetzner Cloud` API plugin for automatic DNS validation.

```sh
curl -O https://raw.githubusercontent.com/acmesh-official/acme.sh/master/acme.sh

# ToDo: Change mail address
sh acme.sh --install \
--home /var/lib/acme/ \
--config-home /etc/acme/ \
--cert-home /etc/acme/certs/ \
--accountemail mail@example.com \
--log /var/log/acme.log

curl -L -o /var/lib/acme/dnsapi/dns_hetznercloud.sh https://raw.githubusercontent.com/acmesh-official/acme.sh/master/dnsapi/dns_hetznercloud.sh

curl -L -o /var/lib/acme/notify/ntfy.sh https://raw.githubusercontent.com/acmesh-official/acme.sh/master/notify/ntfy.sh

source ~/.bashrc
```

After installing, we configure notifications with `ntfy`.

```sh
# ToDo: Change domain, topic and token
export NTFY_URL="https://ntfy.example.com"
export NTFY_TOPIC="acme"
export NTFY_TOKEN="<token>"

# ToDo: Change server name
/var/lib/acme/acme.sh --set-notify --notify-hook ntfy --notify-level 3 --notify-mode 1 --notify-source server-name
```

Now, issue a new SSL certificate using DNS validation with your `Hetzner Cloud` API token.

```sh
# ToDo: Change token
export HETZNER_TOKEN="<token>"

# ToDo: Change domain name (2x)
/var/lib/acme/acme.sh --issue --dns dns_hetznercloud -d example.com -d '*.example.com'  --server letsencrypt --ecc

# ToDo: Change domain name (3x)
/var/lib/acme/acme.sh --install-cert -d example.com --ecc \
--fullchain-file /etc/nginx/certs/example.com/fullchain.pem \
--key-file /etc/nginx/certs/example.com/key.pem \
--reloadcmd "systemctl reload nginx"
```

Finally, add the following SSL certificate paths to your Nginx server configuration.

```sh
ssl_certificate /etc/nginx/certs/example.com/fullchain.pem;
ssl_certificate_key /etc/nginx/certs/example.com/key.pem;
```
