---
title: "DeepLife - IT instructions"
layout: textlay
sitemap: false
permalink: /it
---
# Using the Shared GPU Infrastructure

## Available Infrastructure

In total we will be able to use 2 workstations with 80GB of RAM and 8vCPU cores. Each station has a NVIDIA V100 GPU with 32GB of memory. These workstations are shared by all teams, so please be mindful of the compute needed by your scripts. All workstations come with CUDA v11.0 pre-installed, so you should be able to use the GPUs in your script without needing to install any drivers.

Each Node currently comes with 100GB of disk space which can be extended if more space is needed to fit your data. However due to the limitations in total space we can provide as of now. Please consider using the same raw datasets as other teams working on the same project.

The nodes are not configured as a cluster so you will only be able to use one GPU at a time.

### Future Updates

**More Workstations:** In total we got access to 6 of the workstations described above and are waiting for the final provisioning of those resources, meaning they should be available by the end of this week.

**Shared Storage:** As of now each workstation has a separate Block Storage which means you should keep using the same instance to avoid having copies of data on multiple machines. However we already asked for the provisioning of shared storage instances which should also be available by the end of this week.

Keep an eye on this site for updates to the available infrastructure.

## Accessing the Servers

For access to the servers you will receive an email specifying your username and a password as well as the names of the instances you will have access to. To log in use the following command:

```bash
ssh 'YOUR_USERNAME:INSTANCE_NAME@deeplife.tjh28.com' -p 2222
# Instance names will be denbi1, denbi2, ...
```

After this command you will be prompted for your password, then you should be connected to your instance.

Always use port 2222 for connecting, as the server is configured to only accept incoming connections through this port.

If you prefer to connect with ssh key pairs, generate an ssh key pair and send the public key to tim.hudelmaier@stud.uni-heidelberg.de along with your assigned user name so I can add the ssh key to your account.

## Using the Servers

If you are working with the servers there are 3 key elements:

1. You can connect to the server to add and edit your code using VSCode
2. Terminal connections should be managed using TMUX so whenever you loose connection to your terminal it is not stopped.
3. You run your code using Docker to isolate your runtime environment.

## Storing Data

Currently only one volume is mounted on each machine. You can find this volume under:

```bash
cd /mnt/volume
```

Please create a folder for your project and within the project folder sub-folders for each group working on that project to make sharing data between groups easier and keeping the volume clean.

**All files, code and data, for your project should be stored on this volume!** As the volumes are persistent and can be attached and detached from different machines. Avoid storing data on the machines disk at all cost as it is only 20GB in size.

Later you will need the path to your projects folder to mount it as a volume to your docker container. You can get the path by cd-ing into your project directory and running:

```bash
pwd
```

### Connecting with VSCode

Follow these steps to connect to your server on VS Code:

1. CMD + SHIFT + P > Remote-SSH: Connect to Host > Add new SSH Host ...
   ![img](./images/connect_shh.png)
2. Paste the command you use to connect via ssh (change _username_ and _denbiX_ to match your user name and instance)
   ![img](./images/ssh_command.png)
3. Select a SSH config to update
   ![img](./images/select_config.png)
4. Open your SSH configuration file
   ![img](./images/open_config.png)
5. And edit the entry in the following way:

```
Host give_your_host_a_name
	HostName deeplife.tjh28.com
	User username:denbiX
	Port 2222
```

In case you have set up a public key, you can add the following line to ensure it is used:

```
IdentityFile ~/path/to/you/private/key
```

This will likely look something like this: `~/.ssh/key_name`

6. Then you can finally connect to your host by running **CMD + SHIFT + P > Remote-SSH: Connect to Host** and selecting your HostName. After that you will be prompted for your password. After that you are connected to the workstation!
   ![img](./images/enter_password.png)

### TMUX

The usage of TMUX is very simple. The first time you use the terminal you can create a new session by running:

```bash
tmux new -s session_name
```

Tmux will then start automatically. You can detach from your session by running:

```bash
tmux detach
```

If you want to attach to your session again at a later point use:

```bash
tmux a -t session_name
```

a great TMUX cheat sheet can also be found here: https://tmuxcheatsheet.com

### Using Docker

To isolate each projects runtime we will be using Docker. To get your relevant dependencies you can pull pre-made Docker images (with pytorch etc. already installed) from the Docker Hub.

First check if the image your want to use is already available:

```bash
docker images
```

If no already installed image matches your requirements your can pull an image using:

```bash
docker pull pytorch/pytorch
```

Finally start your container using

```bash
docker run --gpus all --rm -it -v /mnt/volume/path/to/your/project/:$HOME pytorch/pytorch /bin/bash
```

this will start a container and push you to the bash of that container so you can now call and execute your scripts.

The flag `-rm` is optional. If this flag is set, then the container will automatically be deleted after you exit from the container.

The `-v`flag is used to attach a volume (your project directory) to the container so it can access your scripts and your data.

Do not forget to use tmux when working with your containers to avoid the container closing and loosing your progress.

**Please do not install python and try to run scripts on the workstation itself! Always use Docker.**

You can check if your container is running as follows:

```bash
docker ps
```

### Using Devcontainers

If you prefer to work on the workstation as you would on your local machine (i.e. using jupyter notebooks and debugging in VSCode) you can also create a devcontainer.

This is also just a docker container you can connect to within VSCode so all code you run will be run inside this container, again isolating your environment.

To use devcontainers within your team sub-folder create a directory: `.devcontainer`

```bash
sudo mkdir .devcontainer
```

Within the directory create a file: `devcontainer.json`

```json
{
  // Set the name of the devcontainer
  "name": "projectX-groupXYZ",

  // // VSCODE installs a vscode-server on top of the created custom image.
  "image": "pytorch/pytorch:latest",

  // Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
  // "build": {
  // Path is relative to the devcontainer.json file.
  // "dockerfile": "Dockerfile"
  // },

  // Specify under which name the devcontainer should be run and which GPUs to use
  "runArgs": ["--name", "dev_yourname", "--gpus", "all"],

  // Specify where workspace directory is mounted in the devcontainer
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "workspaceFolder": "/workspace",

  // Add vscode extensions (they will be installed to the container home directory)
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-toolsai.jupyter", // enable Python Interactive Window with (#%%)
        "ms-python.black-formatter"
      ]
    }
  }
}
```

Open your team subfolder in VSCode via: File > Open Folder. This step is important so your devcontainer will use your teams config.

Now you can start to develop in your devcontainer by running: CMD + SHIFT + P > Reopen in Container. VSCode will now take care of the heavy lifting and will start your container for you!
