# Online-Node
How to install an Ethereum online node

## Prequirement

Ubuntu 16.04

## Geth

### Install geth

```console
apt-get install software-properties-common
add-apt-repository -y ppa:ethereum/ethereum
apt-get update
apt-get install ethereum
```

### Running geth in background (replace SERVER_ADDRESS)

For testnet network

```console
nohup geth --testnet --rpc --rpccorsdomain "*" &
```

For main network

```console
nohup geth --rpc --rpccorsdomain "*" &
```

For fast sync

```console
nohup geth --fast --testnet --rpc --rpccorsdomain "*" &
```

See geth output

```console
tail -f nohup.out
```

Check if geth is running

```console
ps ax | grep geth
```

Kill geth

```console
pkill geth
```

## Install and configure Nginx as a HTTPS proxy

### Install Nginx

```console
apt-get install nginx
```

### Config Nginx

```console
nano /etc/nginx/sites-enabled/default
```

Replace by **(replace SERVER_ADDRESS)**

```
server {
	listen 80;
	listen [::]:80;
	return 301 https://$host$request_uri;
}
server {
	listen 443 ssl default_server;
	listen [::]:443 ssl default_server;
	
	#ssl_certificate                 /etc/letsencrypt/live/DOMAIN/fullchain.pem;
	#ssl_certificate_key             /etc/letsencrypt/live/DOMAIN/privkey.pem;
	#ssl_trusted_certificate         /etc/letsencrypt/live/DOMAIN/chain.pem;
	
	location /.well-known {
	      root              /var/www/DOMAIN/;
	}

	location / {
	      proxy_set_header        Host $host;
	      proxy_set_header        X-Real-IP $remote_addr;
	      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
	      proxy_set_header        X-Forwarded-Proto $scheme;

        # Fix the “It appears that your reverse proxy set up is broken" error.
	      proxy_pass          http://localhost:8545;
	      proxy_read_timeout  90;
	}
}
```

Note that the ssl_certificate config are commented. As the certificate is not generated yet, Nginx will not restard if the files doesn't exist. We will uncomment them later.

### Restart Nginx

```console
service nginx restart
```

### Certificate

```console
apt-get install letsencrypt
mkdir /var/www/DOMAIN/
letsencrypt certonly --webroot -w /var/www/DOMAIN/ -d DOMAIN
```

### Edit Nginx config to remove the comment ssl lines

```console
nano /etc/nginx/sites-enabled/default
```

Uncomment the lines like:

```
	ssl_certificate                 /etc/letsencrypt/live/DOMAIN/fullchain.pem;
	ssl_certificate_key             /etc/letsencrypt/live/DOMAIN/privkey.pem;
	ssl_trusted_certificate         /etc/letsencrypt/live/DOMAIN/chain.pem;
```

### Restart Nginx

```console
service nginx restart
```

### Renew certificate auto

```console
crontab -e
```

Add

```console
37 4 * * * letsencrypt renew >/dev/null 2>&1
```

## Source

https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu
https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins
https://certbot.eff.org/#ubuntuxenial-nginx
