# REFERENCE FOR SSH TO REMOTE SERVER

# Setup Guides

get the tools and get them set up for remote hosting.

## TOOLS

- `openSSH` allows ssh connections
- `ngrok` is a free tunneling service, though reserved subdomains start at $5/mo. allows easy sharing of hosted website or webservice by redirecting it to a public URL. we can use it for hosting a jupyter notebook without bothing with `ssh`.
- `PyCharm Professional` student license will allow easy remote access through ssh
- `ddclient` allows IP address changes to be reflected if using a domain name (e.g. `my-domain.com`)
- `TeamViewer` is free screen-sharing and remote-access software. while not ideal for coding with, it does allow to connect to your remote machine and manipulate it with the standard graphical desktop (with some lag). useful for connecting to troubleshoot, etc.
- `screen` allows you to run apps in the background

## JUPYTER NOTEBOOK CONFIGURATION

(on the remote machine) create a jupyter configuration file, if you haven't already:

`$ jupyter notebook --generate-config`

then create a password with:

`$ jupyter notebook password`

see details here: [http://jupyter-notebook.readthedocs.io/en/stable/public_server.html]

## NGROK SERVER SETUP

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

## GET PYCHARM PRO

- use your student email address (snu email tested) at [https://www.jetbrains.com/student/] to get activation key
- while you're at it, go to [https://education.github.com/pack] for more free stuff

# Connection Guides

once you have the above set up, you can start a server, transfer your files and start coding

## CONNECT TO REMOTE MACHINE VIA SSH

you can connect directly via the remote PC's IP address:

`user@local:~$ ssh <user>@123.45.67.89`

if you want to connect via domain, setup dynamic-dns and register DNS with domain
for domains.google.com, see their guide.

then you should be able to `ssh` in wih the domain name:

`user@local:~$ ssh <user>@my-domain.com`

## FILE TRANSFER

### using sftp

connect via `ssh` then open `sftp`:

`user@local:~$ sftp <user>@<remote IP or domain>`

if connecting with a different port, use `-oPort=####`:

`user@local:~$ sftp -oPort=1234 <user>@<remote IP or domain>`

you can move directories as in the usual shell; preprending an 'l' moves around the local machine

```
remote   local    function
pwd      lpwd     get current working directory
cd       lcd      change working directory
ls       lls      list files/folders in current directory (use -la for more info)
df -h    ! df -h  (local is 2 commands, ! then df): check space before transfer
```
moving files:

`$ put filename` : transfer files *from* local working dir to remote working dir
`$ get filename` : transfer files *to* local working dir from remote working dir

ref: https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server

### using scp

this can be used to transfer local files to remote system using `ssh`:

navigate to local directory, and then use:

`user@local:~$ scp filename <user>@<remote IP or domain>:~/`

specify port with `-P 11000` and remote subdirectory at the end, ex:

`user@local:~/Downloads$ scp Readme.md derek@dereks.com:~/project1/`

will send the file `Readme.md` from local `Downloads` directory to `derek/home/project1` on remote machine

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

## USING PYTHON/IPYTHON/VIM OVER SSH

- connect to remote PC via ssh (see above)
- in terminal, enter `python` or `ipython` or `vim`

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

## DEVELOPING IN PYCHARM OVER SSH

- open Pycharm pro on local machine and create a new Pycharm Project project
- for **Interpreter**, hit the gear icon on the right and choose `Add Remote...`
- on the window, choose `SSH Credentials`
- `Host` is your remote IP or domain address
- `User name` is your remote user name
- `Auth type` is password (remote system login password) or key
- `Python Interpreter Path` is your `python` path. this will depend on if you are using eg `anaconda`
- finally you will be asked a save path for the project on the remote machine (files will be transfered there)
- NOTE: to get path to default python, remote into machine and do:

`user@remote:~$ which python` 

...and copy this path into the interpreter path field. for me, it is `/home/derek/miniconda3/bin/python`

# Other commands

## checking system resources with GUI over SSH

`ssh -XY user@remote "gnome-system-monitor"`

# REFERENCES

[https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps]

[https://md.ekstrandom.net/blog/2016/04/remote-analysis-with-jupyter-and-ngrok/]

[https://coderwall.com/p/ohk6cg/remote-access-to-ipython-notebooks-via-ssh]
