# REFERENCE FOR SSH TO REMOTE SERVER

## SUGGESTED FIRST STARTUP STUFF

- install `ngrok`, register paid acct for subdomain reservation
- register domain (e.g. with google domains) and set up dynamic dns with `ddclient`
- install `TeamViewer` on local and remote PCs and link both to your account

## REMOTE INTO REMOTE MACHINE

setup dynamic-dns and register DNS with domain

for domains.google.com, see their guide.

then you should be able to `ssh` in wih the domain name:

`user@local:~$ ssh <user>@<domain>`

## STARTING JUPYTER NOTEBOOK OVER NGROK

### on remote machine, startup jupyter server

use **--port=####** for custom port

`user@remote:~$ jupyter notebook --no-browser`

### startup ngrok instance:

set region to Asia-Pacific, specify subdomain and password-protect
specify jupyter port if needed (default: 8888)

`user@remote:~$ ./ngrok http -region ap -subdomain=<subdomain> --auth 'user:password' 8888`

### on local machine, access with ngrok url in browser

`http://<subdomain>.ap.ngrok.io`

## STARTING JUPYTER NOTEBOOK OVER SSH

### on remote machine, startup jupyter server

use **--port=####** for custom port

`user@remote:~$ jupyter notebook --no-browser` 

### create tunnel with ssh

FIRST 8888 is local port, SECOND 8888 should be remote can change, for example if already running local notebook on 8888

use **-f** flag if don't want to keep the terminal open (will run process in background)

`user@local:~$ ssh -N -L localhost:8888:localhost:8888 user@remote`

### on local machine, access in browser:

`localhost:8888`

### if using **-f**, kill process like this:

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
