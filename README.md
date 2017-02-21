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

Install

```console
apt-get install nginx
```

Certificate

```console
cd /etc/nginx
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.crt
```

Config

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
  
	ssl_certificate           /etc/nginx/cert.crt;
	ssl_certificate_key       /etc/nginx/cert.key;

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

Restart Nginx

```console
service nginx restart
```

## Source

https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu
https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins
