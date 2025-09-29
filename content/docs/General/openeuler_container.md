---
title: "START HERE - Getting started as a participant" 
weight: 10
---

# Getting started as a participant
## Create an SSH Key
For more info, [click here](https://docs.oracle.com/en/cloud/cloud-at-customer/occ-get-started/generate-ssh-key-pair.html) or ask for help
### On Linux/Mac
1. Run `ssh-keygen`
2. Do not set a passphrase
3. Save the key-pair somewhere you can find it
### On Windows
You can use PuTTY to create the key, likewise don't set a passphrase
[See here for information ](https://learn.microsoft.com/en-us/viva/glint/setup/sftp-ssh-key-gen#:~:text=Create%20an%20SSH%20key%20pair%20on%20Microsoft%20Windows)
## Creating a SAFE account
1. Go to [the SAFE website](https://safe.epcc.ed.ac.uk/signup.jsp)
2. Fill out the necessary details
	- **You must use your student email**
3. Upload your SSH **public** key to the form - `*.pub`
4. Press "Register"
5. Check your email for confirmation, then finish registration by clicking the link and setting your password
## Joining the Hackathon Project
1. Go [here](https://safe.epcc.ed.ac.uk/TransitionServlet/ApplyProject//-/Transition=Apply) to request to join the project
2. Look for `eidf219 - InfSoc GPU hackathon`
3. Select that, click `Next` then `Apply`
4. We will approve your application as soon as we can
5. We will then create you a login account for your VM
	1. You'll get another email, set your password and MFA OTP
	2. You should already have Microsoft Authenticator for uni, but others will work
## Getting a VM created
1. We'll aim to create VMs for teams before the day
2. If you form a team on the day, we'll need to create you a VM - so let us know!
## Connecting to your VM
- You can connect to the VM CLI via a web interface (VDI), but we'd recommend using SSH in your terminal.
- To SSH to your VM you must first go through the EIDF gateway as a security measure, so your SSH command will likely take the form:
	- `ssh-add /path/to/ssh-key && ssh -J [username]@eidf-gateway.epcc.ed.ac.uk [username]@[vm_ip]`
	- That is, your project account username
- The [EIDF provides documentation](https://docs.eidf.ac.uk/access/ssh/) on connecting via SSH - following this is best, if you're struggling ask an organiser for help
- When prompted for your "OTP" enter the number from your MFA app.
- You should then see a shell inside your VM
## Spawning an openEuler container
**You can follow these instructions on any Linux device, including the provided VM, or even your laptop**. If you're running Windows, we'd highly recommend checking out [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/setup/environment).
1. Put the following into a file called `Dockerfile`:
```Dockerfile
FROM openeuler/openeuler:latest

SHELL ["/bin/bash", "-c"]
RUN /usr/bin/dnf update -y
RUN /usr/bin/dnf install -y python3 nodejs npm curl go sqlite htop
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
RUN . "$HOME/.cargo/env"
RUN /usr/bin/dnf group install -y "Development Tools"

WORKDIR /work

ENTRYPOINT /bin/sh
```
2. Put the following into a file called `docker-compose.yaml`:
```yaml
services:
  oe:
    build : .
    container_name: "oe"
    tty: true
    stdin_open: true
    volumes:
      - ./work:/work
```
3. Connect your VM
4. Start the container with `docker compose build && docker compose up -d`
5. Open a shell in the container with `docker exec -it oe bash` - it should have several tools pre-installed
