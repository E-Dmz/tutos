# How to run tasks from anywhere
> As a student in Le Wagon `data science` batch #722 (Paris, Oct.-Dec. 2021), i had a great experience remotely running tasks on my *home desktop* from my laptop *on campus*. 
>
> In particular, it can be 10 times faster on `Week 7` and `project`, when you build a Docker image, download or upload big files and on `Week 6` and `project`, when you train a neural network.
>
> I'd like to share the few tricks I learnt from scratch because i think it might be of interest for some students in future batches and fellow freelancer alumni.

#### How you can help...
- ...improve this content by giving your feedback or suggest additions
- ...share this to students if you think it can help
- ...tell me what instructions are missing, what is not clear enough and if you try it, what does not work
- I litterally started from zero knowledge, so i will really appreciate your feedback when i misunderstand something or if you know a better way.
- I'm missing some steps in SSH server/client setup (and also on *Optional WOL config*). If you start doing it from scratch, please note all the steps so that we can have the whole story
- I don't know how it transposes to other OS
- I give credit to our TA Yassine for showing me the magic of port forwarding

#### About my setup
- my desktop (2021) is a Linux Ubuntu with AMD Ryzen 5, 16 Gb RAM, MSI GeForce GTX 1650
- my laptop (2019) is a Windows Surface Laptop 2 (2019) with Intel Core i5, 8 Gb RAM and 128 Gb ROM
- i also have a NAS server Synology DS720+ (2021) that I use to remotely switch on my desktop (see the `Optional: WOL` section)
- my internet box is a Livebox 3 (ISP Orange)

## SSH Setup on your desktop
- First of all, you need to setup a SSH Client/Server on your desktop: I followed [these instructions](https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows). 
- You also need configured NAT/PAT rules on my Orange Box and change the ssh port (default 22).

## Remote-control your desktop with ssh
- Your desktop must be on 
- Optional: if you want to remotely switch on your desktop, what you need is WOL (Wake On LAN). See the `Optional: WOL` section.
```bash
# ssh login to desktop
ssh <username>@<my_public_IP> -p<ssh_port>
# hurray! now you're on your desktop shell 
```

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

## SSH with Port forwarding 
- With the following command, you ssh to your desktop and redirect a remote (desktop) `source port` to a local (laptop) `destination port`. 
```bash
# ssh login and port forwarding
ssh <username>@<my_public_IP> -p<ssh_port> -L <source_port>:localhost:<destination_port>
```
- using the same port number on both your laptop and your desktop will make your life easier
- if you lose connection or put your laptop on sleep without ending the connection with `Ctrl+D`, strange things will happen and you may need to connect via another port and/or wait a bit before it works.

## Running a Jupyter lab server
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

## Running other servers
You can launch several servers... just SSH multiple time with ditinct port redirection:  
- `jupyter notebook` (with its friendly shortcuts and nbextensions)
- `uvicorn` **(didn't try)**
- `docker run`
- `streamlit` **(didn't test)**

## What you cannot do **(as far as I know)**
- Use VSCode
- `gcloud auth login`, `heroku login`?? **(didn't try)**
- with `heroku` you have a `IP address mismatch` error
- but with `gcloud`, you can authenticate! 

## The real problem with public IPs...
- ...is that from time to time, it is changed by your ISP. 
- i had that bitter experience on the *one but last* day of the project.
- at home, you can always run: `curl ifconfig.me`
- i do not know a way get your public IP from the outside world... you may need DynDNS for this **(didn't try)**
- ...with my NAS server, i can always connect to a smart user interface and run a command that writes the IP address to a file (or have it sent by mail!), which is very handy

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
- I had to set up a "broadband over power line (BPL)" = *"courant porteur de ligne (CPL)"* because you can't WOL with Wi-Fi
- I remember having a hard time setting up WOL. Inspiration came from [this page](https://help.ubuntu.com/community/WakeOnLan) and/or [this one](https://necromuralist.github.io/posts/enabling-wake-on-lan/) and/or one of the other numerous. It may differ from one computer to another, but if you do this from scratch, it's worth noting every step in order to complete that tuto  

## Optional: Environment variables and Makefile
Note: **I didn't test this, and i really don't know if it's worth it**
```bash
# ~/.zshrc on LAPTOP
## (echo "this" >> ~/.zshrc)

##################################
### CONNECT CONFIGURATION
##################################

export USERNAME_DESKTOP=XXXXXX
export PUBLIC_IP_HOME=XXX.XXX.XXX.XXX
export PORT_SSH_DESKTOP=XXXX

alias cx-desktop="ssh ${USERNAME_DESKTOP}@${PUBLIC_IP_HOME}"

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_API=5000
export PORT_STREAMLIT=8000

alias cx-notebook="cx-desktop -L -p${PORT_NOTEBOOK}:localhost:${PORT_NOTEBOOK}"
alias cx-lab="cx-desktop -L -p${PORT_LAB}:localhost:${PORT_LAB}"
alias cx-api="cx-desktop -L -p${PORT_API}:localhost:${PORT_API}"
alias cx-streamlit="cx-desktop -L -p${PORT_STREAMLIT}:localhost:${PORT_STREAMLIT}"

##################################
```

```bash
# ~/.zshrc on DESKTOP
## (echo "this" >> ~/.zshrc)

##################################
### CONNECT CONFIGURATION
##################################

export PORT_NOTEBOOK=8880
export PORT_LAB=8890
export PORT_STREAMLIT=8000
export PORT_API=5000

alias jn-cx="jupyter lab --port=${PORT_NOTEBOOK}"
alias jl-cx="jupyter lab --port=${PORT_NOTEBOOK}"

##################################
```


```Makefile
# Makefile on desktop (streamlit project folder)
run_streamlit_cx:
    streamlit run app.py --server.port ${PORT_STREAMLIT}

# Makefile on desktop (fastapi project folder)
run_api-cx:
    uvicorn my_api.fast:app --reload --port ${PORT_API}
```

