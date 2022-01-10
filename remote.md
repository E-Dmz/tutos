# How to run tasks from anywhere
> As a student at Le Wagon `data science` (batch #722, Paris), i started to run tasks remotely on my ***home* desktop PC** from my ***on campus* laptop PC**. 
>
> I found it could be **10 times faster** for building or pushing a Docker image, for downloading or uploading big files (on `Week 7` and `project`), or for training or running big models.
>
> I'd like to share the few tricks I learnt from scratch, as it might be of interest for students in future batches and fellow freelancer alumni.
>
> Using aliases, connecting to my Jupyter lab is now as simple as [ `cxlab` + `password` + `cxlab` + `Ctrl + click` ].

#### Table of contents 
<!--
move this to blog and/or tuto: 
https://stackoverflow.com/questions/11948245/markdown-to-create-pages-and-table-of-contents/33433098#introduction

AND MAKE A PARSER to automate this MD TOC (can be a 
-->
1. [About the setup i used](#about-the-setup-i-used)
2. [How to setup SSH](#how-to-setup-ssh)
3. [How to connect (SSH)](#how-to-ssh)
4. [What you can do once you're connected](#what-you-can-do-once-youre-connected)
5. [How to connect with port forwarding ](#how-to-connect-with-port-forwarding)
6. [How to run Jupyter lab](#how-to-run-jupyter-lab)
7. [How to run other services](#how-to-run-other-services)
8. [What you cannot do, yet](#what-you-cannot--do-yet)
9. [The issue with public IPs...](#the-issus-with-public-ips)
10. [Optional: WOL](#optional-wol)
11. [Optional: Set aliases, environment variables](#optional-set-aliases--environment-variables)
12. [Optional: Set Makefile, Dockerfile...](optional-set-makefile--dockerfile)
13. [How you can help](how-you-can-help)

...

#### About the setup i used
- desktop PC (2021): a Linux Ubuntu with AMD Ryzen 5, 16 Gb RAM, MSI GeForce GTX 1650
- laptop PC (2019): a Windows Surface Laptop 2, Intel Core i5, 8 Gb RAM and 128 Gb ROM
- Le Wagon's [Data Science Bootcamp Setup](https://github.com/lewagon/data-setup) on both
- i also have a NAS server Synology DS720+ (2021) that I use to remotely switch on my desktop (see the `Optional: WOL` section)
- internet box: Orange Livebox 3

## How to setup SSH
- You need to setup a SSH Client/Server on your desktop PC
- I followed [these instructions](https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows)
- You're welcome to list the precise set of steps if you do it, so that we improve this tuto 
- I also had to configure NAT/PAT rules on the internet box
- It's recommanded to change the SSH port (default 22).

## How to connect (SSH)
- Your desktop must be on 
- Optional: if you want to remotely switch on your desktop, what you need is WOL (Wake On LAN). See the `Optional: WOL` section.
- Just type this command your laptop's Terminal:
```bash
# ssh login to desktop
ssh <username>@<my_public_IP> -p<ssh_port>
# enter your password...
```
- ***Hurray!*** now you're seeing your desktop PC's `Terminal` ***inside*** your laptop PC's `Terminal` 
- (all the fancy formatting of `oh my sosh!` is lost)

## What you can do once you're connected
- anything you're used to do from a terminal
- notably `cd`, `cat`, `curl`, `git`, `heroku`, `docker`, `gcloud`, `pip install`, `shutdown`, etc.
- just remember you're not on your local `Terminal`
- you may want to run multiple `Terminal`s in parallel
- when you're done, don't forget to quit the shell (and close the connection) by pressing `Ctrl-D`
- you no longer have the pretty formatting of `oh my zosh!`
- to run `jupyter lab` or any other service, all you need is: `Port forwarding` **(next section!)**
- my routine is:
    - i ssh to my desktop
    - i launch `jupyter lab` **(next section!)**
    - then i run my commands on `jupyter lab`'s `Terminal`

## How to connect with port forwarding 
- With the following command, you ssh to your desktop and redirect a remote (desktop) `source port` to a local (laptop) `destination port`. 
```bash
# ssh login and port forwarding
ssh <username>@<my_public_IP> -p<ssh_port> -L <source_port>:localhost:<destination_port>
```
- using the same port number on both your laptop and your desktop will make your life easier
- if you lose connection or put your laptop on sleep without ending the connection with `Ctrl+D`, strange things will happen and you may need to connect via another port and/or wait a bit before it works.

## How to run Jupyter lab
```bash
# run jupyter lab and specify the port you've just chosen
jupyter lab --port=<source_port>
```
- if the `<source_port>` and `<distant_port>` have the same number, you just have to use the link displayed in the terminal (url with token) or http://localhost:<destination_port>/lab (may need a password)
- and voilà! **The user experience is very familiar**... as if you were running `jupyter lab` on your laptop
- with `jupyter lab`'s you can:
    - edit and run a notebook
    - navigate in your file system
    - use `Launcher`'s `Terminal` to run commands (`git`) or `Text File` for edition
    - most of all: **you can easily transfer files** ***from*** your desktop (right-click and `Download`) and ***to*** your desktop (drag and drop).

## How to run other services
You can launch several services... just SSH multiple time with ditinct port redirection:  
- `jupyter notebook` (with its friendly shortcuts and nbextensions)
- `uvicorn` **(didn't try)**
- `docker run`
- `streamlit` **(didn't test)**

## What you cannot do **(as far as I know)**
- Use VSCode
- i tried to `heroku login` but there's a `IP address mismatch` error in the browser
- ... but you can `gcloud auth login`, which is pretty cool if you want to push a Docker image to GCP 

## The issue with public IPs...
- ...is that from time to time, it is changed by your Internet Service Provider
- i had that bitter experience on the *one but last* day of the project.
- at home, you can always run: `curl ifconfig.me`...
- i do not know a way get your public IP from the outside world... you may need DynDNS for this **(don't know)**
- ...Synology's `Quickconnect` service always gives access to my NAS so that I can:
    - run the command below, with the NAS Manager's `Task scheduler` > `User-defined script`> `Run`
    - (this writes the IP address to a file)
    - open the file
```bash
echo >> /var/services/homes/edmz/Scripts/IP.txt
curl ifconfig.me >> /var/services/homes/edmz/Scripts/IP.txt
```

## Optional: WOL
- Wake On LAN if you want to remotely switch on your desktop (the motherboard can always recognize a *magic packet* and wake up on this) 
- for some reason, i cannot to do this directly from the outside world: my internet box doesn't allow it or I didn't find the right parameters) 
- but i can WOL my desktop from my local network... 
- ...so what i do is: SSH to my NAS server and WOL from my local network.
```bash
# SSH to my NAS
ssh <username_on_nas>@<my_public_IP> -p<port_to_nas>
# send magic packet to the desktop
sudo synonet --wake <MAC_address> eth0
# Ctrl+D to exit NAS shell
# wait a bit for the desktop to wake up
# SSH to the desktop
```
- You can't WOL over Wi-Fi, you need a physical connection (LAN or *"courant porteur de ligne (CPL)"* - "broadband over power line (BPL)"?) 
- I remember having a hard time setting up WOL. Inspiration came from [this page](https://help.ubuntu.com/community/WakeOnLan) and/or [this one](https://necromuralist.github.io/posts/enabling-wake-on-lan/) and/or one of the other numerous. It may differ from one computer to another, but if you do this from scratch, it's worth noting every step in order to complete that tuto!

## Optional: Set aliases, environment variables
With the following setup, i just need [ `cxlab` + `password` + `cxlab` + `Ctrl + click` ] to connect jupyter lab!

### Add the following lines to .zshrc on your LAPTOP:
```bash
# ~/.zshrc on LAPTOP
## copy-paste this at the end of  ~/.zshrc ***on your LAPTOP PC***

##################################
### CONNECT CONFIGURATION
##################################

export USERNAME_DESKTOP=XXXXXX
export PUBLIC_IP_HOME=XXX.XXX.XXX.XXX
export PORT_SSH_DESKTOP=XXXX

alias cxdesktop="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME} -p${PORT_SSH_DESKTOP}"

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_API=5000
export PORT_STREAMLIT=8000

alias cxnotebook="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME} -p${PORT_SSH_DESKTOP} -L ${PORT_NOTEBOOK}:localhost:${PORT_NOTEBOOK}"
alias cxlab="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME} -p${PORT_SSH_DESKTOP} -L ${PORT_LAB}:localhost:${PORT_LAB}"
alias cxapi="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME} -p${PORT_SSH_DESKTOP} -L ${PORT_API}:localhost:${PORT_API}"
alias cxstreamlit="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME} -p${PORT_SSH_DESKTOP} -L ${PORT_STREAMLIT}:localhost:${PORT_STREAMLIT}"

##################################
```

### Alternatively, use `echo >>`:
```bash
# OR run this:

echo "

##################################
### CONNECT CONFIGURATION
##################################

export USERNAME_DESKTOP=XXXXXX
export PUBLIC_IP_HOME=XXX.XXX.XXX.XXX
export PORT_SSH_DESKTOP=XXXX

alias cxdesktop=\"ssh \${USERNAME_DESKTOP}@\${PUBLIC_IP_HOME} -p\${PORT_SSH_DESKTOP}"

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_API=5000
export PORT_STREAMLIT=8000

alias cxnotebook=\"ssh \${USERNAME_DESKTOP}@\${PUBLIC_IP_HOME} -p\${PORT_SSH_DESKTOP} -L \${PORT_NOTEBOOK}:localhost:\${PORT_NOTEBOOK}\"
alias cxlab=\"ssh \${USERNAME_DESKTOP}@\${PUBLIC_IP_HOME} -p\${PORT_SSH_DESKTOP} -L \${PORT_LAB}:localhost:\${PORT_LAB}\"
alias cxapi=\"ssh \${USERNAME_DESKTOP}@\${PUBLIC_IP_HOME} -p\${PORT_SSH_DESKTOP} -L \${PORT_API}:localhost:\${PORT_API}\"
alias cxstreamlit=\"ssh \${USERNAME_DESKTOP}@\${PUBLIC_IP_HOME} -p\${PORT_SSH_DESKTOP} -L \${PORT_STREAMLIT}:localhost:\${PORT_STREAMLIT}\"

##################################

" >> ~/.zshrc
```
### Add the following lines to .zshrc on your on your DESKTOP
```bash
# ~/.zshrc on DESKTOP
## copy-paste this at the end of  ~/.zshrc ***on DESKTOP PC***

##################################
### CONNECT CONFIGURATION
##################################

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_STREAMLIT=8000
export PORT_API=5000

alias cxnotebook="jupyter notebook --port=${PORT_NOTEBOOK}"
alias cxlab="jupyter lab --port=${PORT_LAB}"

##################################
```

### Alternatively, use `echo >>`:
```bash
# OR run this ***on LAPTOP PC***

echo "

##################################
### CONNECT CONFIGURATION
##################################

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_STREAMLIT=8000
export PORT_API=5000

alias cxnotebook=\"jupyter notebook --port=\${PORT_NOTEBOOK}\"
alias cxlab=\"jupyter lab --port=\${PORT_LAB}\"

##################################
" >> ~/.zshrc
```
## Optional: Set Makefile, Dockerfile...
### Streamlit 
**i haven't try this**
```Makefile
# Makefile on desktop (streamlit project folder)
run_streamlit_cx:
    streamlit run app.py --server.port ${PORT_STREAMLIT}

# Makefile on desktop (fastapi project folder)
run_api-cx:
    uvicorn my_api.fast:app --reload --port ${PORT_API}
```

### in a Docker project
**i haven't try this**

## How you can help
- ...improve this content by giving your feedback or suggest additions
- ...share this to students if you think it can help
- ...tell me what instructions are missing, what is not clear enough and if you try it, what does not work
- I litterally started from zero knowledge, so i will really appreciate your feedback when i misunderstand something or if you know a better way.
- I'm missing some steps in SSH server/client setup (and also on *Optional WOL config*). If you start doing it from scratch, please note all the steps so that we can have the whole story
- I don't know how it transposes to other OS
- I'm thankful to our TA Yassine for showing me the magic of port forwarding
