---
title: "Transfering files between your VM and local machine"
weight: 30
---

## Using GitHub

The first, and perhaps simplest solution is to copy files to your GitHub repository and load them on the VM with git.

Running
```bash
git clone [your_repository_connection] # If you haven't copied the repository to your VM yet
git pull # To update locally from the remote
```

## Using SCP

Next, you can use a one-liner like 
```bash
scp -r -J [your_username]@eidf-gateway.epcc.ed.ac.uk /path/to/local/files/ [your_username]@[vm_ip]:~/
```


