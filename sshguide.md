# REFERENCE FOR SSH TO REMOTE SERVER

# Setup Guide

## TOOLS

- `openSSH` allows ssh connections
- `ngrok` is a free tunneling service, though reserved subdomains start at $5/mo. allows easy sharing of hosted website or webservice by redirecting it to a public URL. we can use it for hosting a jupyter notebook without bothing with `ssh`.
- `ddclient` allows IP address changes to be reflected if using a domain name (e.g. `my-domain.com`)
- `TeamViewer` is free screen-sharing and remote-access software. while not ideal for coding with, it does allow to connect to your remote machine and manipulate it with the standard graphical desktop (with some lag). useful for connecting to troubleshoot, etc.

## NGROK SETUP

this will allow basic access to a remote notebook through an ngrok url.

- setup remote PC with internet access
- install `ngrok` and (optional) register paid account for fixed subdomain
- (optional) install `TeamViewer` on local and remote PCs and link both to your TV account

## SSH SETUP

this will allow a whole slew of options, including access to a notebook through an ssh tunnel.

- setup remote PC with internet access. if behind router, forward port 22 in router settings
- install `openssh-server` on remote PC
- install `openssh-client` on local PC
- use a site like `whatsmyip` to find remote PC's IP address
- (optional) set up ssh keys to skip password login

## CUSTOM DOMAIN SETUP (FOR SSH)

this will let you connect using a domain name instead of an IP address for maximum coolness.

- register domain (e.g. with domains.google.com)
- if your ISP uses dynamic IP addresses, set up dynamic dns with `ddclient`
- NB: if using domains.google.com, you can also update IP through browser address bar with their API

# Connection Guide

## REMOTE INTO REMOTE MACHINE

you can connect directly via the remote PC's IP address:

`user@local:~$ ssh <user>@123.45.67.89`

if you want to connect via domain, setup dynamic-dns and register DNS with domain
for domains.google.com, see their guide.

then you should be able to `ssh` in wih the domain name:

`user@local:~$ ssh <user>@my-domain.com`

## USING JUPYTER NOTEBOOK OVER NGROK

### on remote machine, start a jupyter server

use **--port=####** for custom port

`user@remote:~$ jupyter notebook --no-browser`

### on remote machine, open an ngrok tunnel:

here we set region to Asia-Pacific (`ap`), specify a reserved subdomain and password-protect
we can specify jupyter port if needed (default: 8888)
*do NOT use remote PC's login; this `user` and `password` is just for ngrok*
*replace `<subdomain>` with subdomain of choice*

`user@remote:~$ ./ngrok http -region ap -subdomain=<subdomain> --auth 'user:password' 8888`

### on local machine, access with ngrok url in browser

`http://<subdomain>.ap.ngrok.io`

## USING JUPYTER NOTEBOOK OVER SSH

### on remote machine, start a jupyter server

use **--port=####** for custom port

`user@remote:~$ jupyter notebook --no-browser` 

### create tunnel with ssh

NB: FIRST `8888` is local port, SECOND `8888` should be remote. change these depending on what local and remote ports you want to use. for example, if using `--port=####` for remote notebook, edit the *second* accordingly; and/or if already running local notebook on `8888`, change *first* port to allow access to both notebooks locally.

use **-f** flag (e.g. `-N -f -L`) if don't want to keep the terminal open (this will run the ssh process in the background)

`user@local:~$ ssh -N -L localhost:8888:localhost:8888 user@remote`

### on local machine, access notebook in browser:

`localhost:8888`

### if using **-f**, kill the ssh tunneling process like this:

where `8888` is the LOCAL port:

```
user@local:~$ ps aux | grep localhost:8888

user 12345  0.0  0.0  41488   684 ?        Ss   17:27   0:00 ssh -N -f -L localhost:8888:localhost:8889 user@remote
user 11111  0.0  0.0  11572   932 pts/6    S+   17:27   0:00 grep localhost:8889

user@local:~$ kill -15 12345
```

# REFERENCES

https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
https://md.ekstrandom.net/blog/2016/04/remote-analysis-with-jupyter-and-ngrok/
https://coderwall.com/p/ohk6cg/remote-access-to-ipython-notebooks-via-ssh
