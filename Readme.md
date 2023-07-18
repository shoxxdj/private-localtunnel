# Private localtunnel

## Why ? 

One day I discovered the ngrok project, and I found it to be very cool. My main problem with it was the impossibility to have a self hosted server. 

Then I discovered the localtunnel project on which ngrok was based. 

And, my new problem was to host a server that may be used by anyone without restriction. 

So I made this fork to have restrictions. 

_If you fork this project because it create a problem for you that you wan't to solve let me know :D_

## How ? 

### Components 
As the original project this one is made of 3 layers : 

#### The Server 
The server is responsible of create and manage the localtunnels.
It also generate a secret key that have to be sent to the user for it's authentication.

#### The Client 
The client is the software located on the computer that want to expose some content on the internet without considering firewall issues. 

To be fully functionnal it needs to use the same parameters as the server for authenication.

#### Ngnix Proxy
_Because every good project needs an nginx._

Used for SSL/TLS. You can avoid it if security is not your problem. 

### Authentication 

The authentication is based on a TOTP mechanism.
Client and server shares their configuration and so they are able to generate the same token at a given time.

The client generate a TOTP, and send it through a dedicated header when requesting for a new localtunnel. It also send a timestamp but only for debug as this information is never used on the server side. 
Then the server receive the information and also compute a TOTP, if both values are equals the process continues. 

### Localtunnel 

Well for this part you have a lot of documentaiton on the internet.

## Getting started !

### Server Side 

First, on a VPS git clone the following project : 

```
https://github.com/shoxxdj/localtunnel-private-server
```
Then you can generate the docker container associated and launch it like this : 

```
docker run -d     --restart always     --name localtunnel     --net host    server --port 3000 --domain tunnel.topdomain.tld
```

you can docker inspect this new container and copy the JSON informations printed. Save them for later.

You can now setup your DNS. 

You need for example : 

A tunnel.topdomain.tld => points to IP 

CNAME *.tunnel.topdomain.tld => points to tunnel.topdomain.tld 

If you use Cloudflare do not use their proxy here.

Then generate your SSL/TLS certificates with certbot on your vps 

```
certbot certonly --manual --preferred-challenges=dns --email you@topdomain.tld --agree-tos -d *.tunnel.topdomain.tld -d tunnel.topdomain.tld
```

note that there are 2 domains, one with the wildcard and one with the root domain, it will allow your tunnels to be encrypted and secure the connexion between your client and the server

move the generated files to a directory named ssl in your home ( or where you want but change the docker run command )

```
cp /etc/letsencrypt/live/tunnel.topdomain.tld/fullchain.pem server.crt
cp /etc/letsencrypt/live/tunnel.topdomain.tld/privkey.pem server.key
```

Finaly you can lauch the nginx docker ! 

```
docker run -d    --restart always     --net host     -v $HOME/ssl:/etc/nginx/ssl/     defunctzombie/localtunnel-nginx:latest
```

### Client side 

On the client side install this project : 

```
https://github.com/shoxxdj/localtunnel-private
```

By simply doing : 

```
sudo npm i localtunnel-private -g
```

Then, copy the content of this file : 
```
https://github.com/shoxxdj/localtunnel-private/blob/master/config/default.json
``` 
To ~/.config/localtunnel/config

And complete with the previously obtained data from the server for the TOPT variable, also change the host to tunnel.topdomain.tld

You may now have something like this : 

```
{
  "server": {
    "host": "https://tunnel.topdomain.tld"
  },
  "totp": {
    "secret": "SVRHMEEFYBYOVFNZ3WVHKMFNRA4IEHPG",
    "len": 15,
    "alg": "SHA-512",
    "period": 60
  },
  "headers": "User-Agent=Axios"
}
```

Finaly you can just launch the following : 

``` lt --port 1234 ```

Your localport 1234 is now exposed on the internet well done ! 

### FAQ 

If you have any questions I will resume them here and reply 

### Contacts 

Everything is on my github page. 