# acme.sh

This document is a collection of important information from the original `acme.sh` [documentation]( https://github.com/acmesh-official/acme.sh/wiki).

## Content

- [Installation](#installation)

## Installation

> ℹ️ **Information:** The following instructions use `Hetzner DNS` for validation. Check [here](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) for your provider.

First, create the necessary directories for storing SSL certificates and acme.sh configuration files.

```sh
# ToDo: Change domain name
sudo mkdir -p /etc/nginx/certs/example.com
sudo mkdir -p /var/lib/acme/dnsapi
sudo mkdir -p /var/lib/acme/notify
sudo mkdir -p /etc/acme
```

Next, switch to root user.

```sh
sudo -i
```

After switching, download and install acme.sh along with the Hetzner DNS API plugin for automatic DNS validation.

```sh
curl -O https://raw.githubusercontent.com/acmesh-official/acme.sh/master/acme.sh

# ToDo: Change mail address and domain
sh acme.sh --install \
--home /var/lib/acme/ \
--config-home /etc/acme \
--cert-home /etc/nginx/certs/example.com \
--accountemail mail@example.com \
--log /var/log/acme.log

curl -L -o /var/lib/acme/dnsapi/dns_hetznercloud.sh https://raw.githubusercontent.com/acmesh-official/acme.sh/master/dnsapi/dns_hetznercloud.sh

curl -L -o /var/lib/acme/notify/ntfy.sh https://raw.githubusercontent.com/acmesh-official/acme.sh/master/notify/ntfy.sh

source ~/.bashrc
```

First, we configure notifications with ntfy.

```sh
export NTFY_URL="https://ntfy.example.com"
export NTFY_TOPIC="acme"
export NTFY_TOKEN="{YOUR_TOKEN}"

/var/lib/acme/acme.sh --set-notify --notify-hook ntfy --notify-level 3 --notify-mode 1 --notify-source example.com
```

Now, issue a new SSL certificate using DNS validation with your Hetzner Cloud API token.

```sh
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