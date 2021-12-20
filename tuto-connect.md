# A tuto to run tasks from anywhere
> As a student in batch #722 (Paris, Oct.-Dec. 2021), i had a great experience remotely running tasks on my *home desktop* from my laptop *on campus*. 
>
> In particular, building a Docker image, downloading or uploading big files and training a neural network can be 10 times faster
>
> I'd like to share the few tricks I learnt because it might interest some students in future batches or fellow freelancer alumni
#### How you can help...
- ...sharing this to students if you think it can help
- ...making comments to improve this content
- I litterally started from zero knowledge, so i will really appreciate you feedback id i'm missing something or if you know a better way
- I'm missing some steps in SSH server/client setup (and also on *Optional WOL config*). If you're doing that setup from scratch, please note all the steps so that we can have the whole story
- having this for another system
- there's potentially further development to be made
- I give credit to our TA Yassine for helping me a lot on this
#### About my setup
- my desktop (2021) is a Linux Ubuntu with AMD Ryzen 5, 16 Gb RAM, MSI GeForce GTX 1650
- my laptop (2019) is a Windows Surface Laptop 2 (2019) with Intel Core i5, 8 Gb RAM and 128 Gb ROM
- i have a NAS server Synology DS720+ (2021) that I will mention in the `Optional WOL` section
- my internet box is a Livebox 3 (ISP Orange)

## SSH to your desktop
- First, your desktop must be on (see *Optional: WOL* section to know how i remotely switch it on)

```bash
# ssh login to desktop
ssh <username>@<my_public_IP> -p<ssh_port>

# Ctrl-D if you want to exit desktop shell
```

To achieve this, you first to setup a SSH Client/Server: I followed [these instructions](https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows). I also configured NAT/PAT rules on my Orange Box and change the ssh port (default 22).

Hurray! Now you can:
- do anything you're used to do from a terminal
- notably `cd`, `cat`, `curl`, `git`, `heroku`, `docker`, `gcloud`, `pip install`, `shutdown`, etc.
- just remember you're not on your local `Terminal`
- you may want to run multiple `Terminal`s in parallel
- I usually launch `jupyter lab` and then run commands on `jupyter lab`'s `Terminal`
- to run `jupyter lab` or any other server, all you need is: *Port forwarding* (next section) 

## SSH with Port forwarding 
With the following command, you both connect to your desktop and redirect one desktop's port to one of your laptop's port. 
```bash
# ssh login and port forwarding
ssh <username>@<my_public_IP> -p<ssh_port> -L <source_port>:localhost:<destination_port>
```
- `source port` is remote and `destination port` is local
- using same port number for both of them will make your life easier
- if you lose connection or put your laptop on sleep without ending the connection with `Ctrl+D`, strange things will happen and you may need to connect via another port and/or wait a bit before it works.

## Running a Jupyter lab server
```bash
# run jupyter lab and specify port 
jupyter lab --port=8180
```
As the local and distant port numbers are the same you can use the link or http://localhost:8180/lab directly

With `jupyter lab`:
- experience is identical when you're on a distant or local computer
- with `jupyter lab`'s `Launcher`:
    - you can use `Terminal` to run commands (`git`)
    - you can use `Text File` for edition
- most of all: you can easily transfer files from and to the distant computer

## Running other servers
You can do several port redirection to launch different servers 
- `jupyter notebook` (that i sometimes for its shortcuts and overall UX)
- `uvicorn` **(didn't test)**
- `streamlit` **(didn't test)**

## The problem with public IPs...
- ... is that they can change.
- from your local network you can run: `curl ifconfig.me`
- I do not know a way to know a public IP from the outside world... you may need DynDNS for this **(didn't test)**
- with a NAS remote user interface you can run a command that writes the IP address to a file **(didn't test)**

## Optional: WOL
- Required only if you want to remotely switch on your desktop. 
- For some reason I did not manage to do this directly so I'm connecting to my NAS server so that I can do WOL from my local network.
```bash
# login to NAS
ssh <username>@<my_public_IP> -p<port_to_nas>
# enter password
# wake up the desktop
sudo synonet --wake <MAC_address> eth0
# Ctrl+D to exit NAS shell
```
- I had to set up a "broadband over power line (BPL)" (*courant porteur de ligne*) connection as WOL needs LAN
- I remember having a hard time setting up WOL. Inspiration came from [this page](https://help.ubuntu.com/community/WakeOnLan) and/or [this one](https://necromuralist.github.io/posts/enabling-wake-on-lan/) and/or one of the other numerous


## Optional: Environment variables and Makefile
Note: **I didn't test this**
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

