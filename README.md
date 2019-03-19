# christian-server-setup

- [Login to a server](login)
- [Setup a server](setup)
- [Using tmux](tmux)
- [Using git](git)
- [Using docker](docker)
- [Coding remotly](remote-code)


A simple document to ease the use of Christian's group computing servers.

*Note*: You can always look at the usage of all the commands I will be using with `man <command>` or `<command> --help`

## <a name="login"></a> How to login to a server

You first need to have an account on the server. Your username on all servers is your `<idul>`.

You can first login into a remote machine with `ssh` which will provide secure encrypted communication between two hosts. Here is the command:

`ssh <idul>@<servername>.gel.ulaval.ca`

You will be asked to enter your password, it's the same you always use with your `<idul>`. So far, we have two compute servers, so `<servername>` must be replace with either: **bersimis** or **mitis**.

## <a name="setup"></a> How to setup the server

### Using scp to copy anything on the remote machine

Basic usage of `scp` is

`scp <SOURCE> <DESTINATION>`

If you want to copy an entire directory, you need to add the `-r` flag like `scp -r <SOURCE> <DESTINATION>`.

Here are some concrete example of `scp`:

- **Copy all files with a specific extension in a folder from my computer to the remote machine**: `scp /path/to/folder/*.txt <idul>@servername.gel.ulaval.ca:/path/to/folder/`

- **Copy directory and all sub-directories from remote machine to my computer**: `scp -r <idul>@servername.gel.ulaval.ca:/path/to/folder /path/to/foler`

- **Copy files between two hosts machine (eg. from __bersimis__ to __mitis__)**: The easiest way to do that is ssh in one of the host and then use scp from there.

**Notes:** If you encounter _user unknown_ error while trying to scp between twos hosts, you first need to add the remote host to the known user list (thoses known hosts can be seen in `~/.ssh/known_hosts`). It will be automatically done by trying to `ssh` from within one host to the other. For exemple, start by ssh into __bersimis__, then, from __bersimis__, ssh into __mitis__.

### Copy your shell configuration

If your are connected to the remote machine and you freak out because it is pretty dark and white in there, do not worry... If you already have a nice bloated shell configuration on your pc, an easy thing to do is transfer it to the remote machine. To do so, you can use secure copy command: `scp` which work almost as `cp`.

Personnally, I have my configurations/aliases/functions in my `~/.bashrc` file. So I will copy this file.

`scp ~/.bashrc <idul>@<servername>.gel.ulaval.ca:/gel/usr/<idul>/.bashrc`

If you ssh with a command (`ssh -c`), it will create a non-interactive shell, which won't load `~/.bashrc` but the non-interactive shell `~/.profile` instead. To overcome this and make sure you always get the same shell configurations, you can run the following on the remote machine.

`echo "source ~/.bashrc" >> ~/.profile`

Some things to point out here about the server: your `$HOME` directory will always be `/gel/usr/<idul>` and `/home-local/<idul>` (both link to the same place). There is extra storage available at `/home-local2/<idul>.extra.nobkp/` and putting your large data and results files there is greatly encourage.

## <a name="tmux"></a> Using tmux, the path to an efficient journey

[tmux shortcuts&cheatsheet](https://gist.github.com/MohamedAlaa/2961058)

Suppose you want multiple terminals on the remote machine, you would need to ssh multiple times. This is not ideal. Furthermore, if you ssh in, start a training (eg. `python train.py`) and then lose your internet, it will surely disconnect your ssh, killing your terminal on the remote machine and stopping your training. This you sure do not want ever to happen.

One way to evercome that is to use a terminal multiplexer, eg. `tmux`.

`tmux`is great :D. You can install it on your machine with `sudo apt-get install tmux`. It is already installed on __bersimis__ and __mitis__.

So basically, `tmux` allows you to have multiple terminals in one terminal. To start a tmux session, you can just type `tmux` in the terminal. But it is preferable to give it a name, therefore, you can do the following:

`tmux new -s <session-name>`

Once you are in your brand new tmux session, you can split vertically or horizontally your terminal into two terminals with `ctrl-b`+`"` and `ctrl-b`+`%`.

If you want a new full size terminal into your tmux session, you can do `ctrl-b`+`c` and use `ctrl-b`+`n` to navigate between them.

To close subterminal, you can do `ctrl-d` or `ctrl-x` to kill it.

One very nice tool is the option to _detach_ from a tmux session with `ctrl-b`+`d`. Doing so, you will exit you tmux session, but everything in it will keep running as if you were still connected to it.

You can _attach_ back into your tmux session with `tmux a -t session-name`

You can also list all your current tmux sessions with `tmux ls`.

You can do `ctrl-d` to exit a tmux session (this tmux session will no longer be available).

## <a name="git"></a> Using git/github

git is already installed on __bersimis__ and __mitis__. To setup your account correctly, you just need to add the username and email of your github account.

`git config --global user.name <USERNAME>`

`git config --global user.email <EMAIL>`

## <a name="docker"></a> Using docker & nvidia-docker

On __bersimis__ and __mitis__, it is expected to run everything in a docker container.

I will not go into details on what docker is, instead I will just write out my view of it. There is 4 things:

* __docker__: This is just the name of the thing, nothing more.

* __dockerfile__: This is a file describing every aspect of a computer on the software level. With this file you know exactly the state of a machine (the environment).

* __image__: An image is the actual computer (or instance) describe in a dockerfile. It is not a description of the computer, it is actually containing every library needed to run it.

* __container__: You might have an image build, but it is doing nothing for the moment, you need to _start_ (or _run_) the computer (the image) and give it a specific command to execute. Doing so, you have a container. An image running a command is a container.

Here are more specific explanations for each point.

**dockerfile**: Since it is quite hard to know every libraries needed for a os to run, you will most likely start your _dockerfile_ with the _FROM_ command. This allow you to build _dockerfile_ on top of the other. Here is an example of a _dockerfile_ used for _tensorboard_.

```python
FROM tensorflow/tensorflow

# Export port for TensorBoard
EXPOSE 6006

WORKDIR /tensorboard

# Start running tensorboard when container is launched
ENTRYPOINT ["tensorboard", "--logdir=/tensorboard/"]
```

** Generating _tensorboard_ events is pretty easy with _tensorflow_. To generate _tensorboard_ events with _Pytorch_ you need to install [TensorboardX](https://github.com/lanpa/tensorboardX)([doc](https://tensorboardx.readthedocs.io/en/latest/index.html)).


All my _dockerfile_ are in the same directory on __bersimis__ and __mitis__, for say `/path/to/dockerfiles/tensorboard.docker`

**image**: In order to have an image, you have to build it with

`docker build -f tensorboard.docker --name tensorboard-gab .`

Run this command while being in the folder `/path/to/dockerfiles/`, the `.` is important at the end of the command.

You can list all docker images with `docker images`

**container**: Once the image is build, you can use the command `docker run image-name` to run it. If you want your image to easily have acces to GPU ressources (which is nice when training networks), you should use `nvidia-docker docker run image-name`

You can list all running container with `docker container ls`. If you add the flag `-a` you can see containers that are not running.

`docker run` comes with a lot of options and flags (`docker run --help`). Here are some of the one I use:


`NV_GPU=0 nvidia-docker run --rm -it -v "/path/to/dir:/mnt/dir" --user $(id -u) --shm-size=10g --name container-name image-name`

* __NV_GPU__ isolate the container to a specific GPU.

* __--rm__ Remove the container when it is close. I always use this flag when I run a container.

* __-it__ This actually two flags (__-i__ and __-t__). This one will create a terminal in the container and will make interactive, so you can navigate interactively in the container.

* __-v__ This is very nice, it allows you to mount one of your folder in the container. You can use this option multiple times.

* __--user__ Without this, the container starts and gives you root permission. This is not actually very clever if your container write file on your computer, you won't be able to modify them. I always start container with `--user $(id -u)` which will give me the same permissions as on the remote machine.

* __--shm-size__ If you have memory error while training, try allowing more memory to the container with this option.

* __--name__ Simply gives a name to the container.

* __-w__ Start the container in a specific working directory.

* __-d__ Giving this flag will start the container and _detach_ from it right after.

**Detaching and attaching from/to a container**: You can detach from inside a running container with the combination `ctrl + p, ctrl + q`. You can than reattach to it with `attach`. Here is an example: 

`docker attach container-name`


**Listing the running container**: It is very useful to see what are you currently running with `ps`. Here is an example:

`docker ps -q --filter "name=pattern"`

- `-q` → quiet, affiche seulement l'id des containers
- `--filter`  → filtre la requête
    - `"name=pattern"` → filtre en fonction des noms des containers

**Killing a container**: It is also interesting to kill containers with `kill`, though pay attention with that, you have the permission to kill others' containers. Here is an example of powerful command nesting in shell to kill a list of container from a name pattern:

`docker kill $(docker ps -q --filter "name=pattern")`

#### Useful aliases/functions
Starting container can rapidly get cubersome. Here is two shortcuts I am using.

`alias doc='nvidia-docker run -it --rm --user $(id -u) --shm-size=10g -v /home-local/galec39.nobkp/path/to/code/:/workspace/code/ -v /home-local2/galec39/path/to/data/:/workspace/data -v /home-local2/galec39/path/to/results:/workspace/results/ -w /workspace/code/'`

With this alias, i can juste do `NV_GPU=0 doc --name container-name image-name` to start container.

The dockerfile above is the one I am using for tensorboard only. To run it, I made a function in my `~/.bashrc` file:

```shell
function db {
port=6006
volume='.'
name=tensorboard-$RANDOM
OPTIND=1
while getopts 'p:n:v:' flag; do
  case "${flag}" in
    p) port="${OPTARG}" ;;
    n) name="${OPTARG}" ;;
    v) volume="${OPTARG}" ;;
    ?)
      echo "script usage: $(basename $0) [-p port] [-n name] [-v volume]" >&2 ;;
  esac
done
OPTIND=1

docker run --rm -d --user $(id -u) -p $port:6006 -v $(readlink -f $volume):/tensorboard --name $name tensorboard-gab
}
```
(Dont forget to `source ~/.bashrc` after each modifications of your `.bashrc` file.)

This function use the image `tensorboard-gab` already build on __bersimis__ and __mitis__. What it does is start the container, start tensorboard and detach from it.

So now, suppose that you have _tensorboard_ events in the folder `/path/to/events/`

You can just do `db -p 6006 -v /path/to/events -n container-name`

Simple as that! To close this  container, you have two options: kill it, or attach to it and close it.

## <a name="remote-code"></a> How to code remotly from my laptop

Here is how I can easily code in a docker on a remote machine. To do this, I use `sshfs`.

First of, I recommend you to create the directory `~/mnt/` which will be use to mount on different remote machine. For each remote machine you want to work on, make a directory of that remote machine in `~/mnt/`. For instance, I can work on __orleans__, __bersimis__ and __mitits__. So I have directories `~/mnt/orleans/`, `~/mnt/bersimis/` and `~/mnt/mitis/`. Then, i put all my code on the remote machine (with git) and I mount the directory (eg `/gel/usr/galec39/path/to/code/`) to the folder in `~/mnt/` corresponding to the remote machine I want to code.

`sshfs -p 22 -o uid=$UID <idul>@bersimis.gel.ulaval.ca:path/to/code/ $HOME/mnt/bersimis -o auto_cache,reconnect,default_permissions`

You should then find your code in `~/mnt/bersimis/code`. Every modifications on your laptop will be sent on the remote machine.

Now, you can create a container and mount the folder `/gel/usr/galec39/path/to/code/` in the container (with the `-v` flag).

So your container, on the remote machine, will contains the code you are modifying on your laptop. You can now use the interactive terminal in the container to execute your code (eg. `python train.py`)

When you are done, you have to _unmount_ `~/mnt/bersimis` by doing:
`fusermount -u ~/mnt/bersimis` on your laptop.










