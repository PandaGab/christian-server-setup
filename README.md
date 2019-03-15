# christian-server-setup

A simple document to ease the use of compute server.

*Note*: You can always look at the usage of all the commands I will be using with `man <command>` or `command --help`

## How to loggin

You first need to have an account on the server. Your username on all servers is your `<idul>`.

You can first loggin into a remote machine with `ssh` which will provide secure encrypted communications between two hosts. Here is the command:

`ssh <idul>@servername.gel.ulaval.ca`

You will be asked to enter your password, it's the same you always use with your `<idul>`. So far, we have two compute servers, so `servername` must be replace with either: **bersimis** or **mitis**.

## Copy your shell configurations

You then entered into the remote machine. It is pretty dark and white in there... One easy thing to do is transfer your shell configurations into the remote machine. To do so, you can use secure copy command: `scp` which work almost as `cp`.

Personnally, I have my configurations/aliases/functions in my `~/.bashrc` file. So I will copy this file.

`scp ~/.bashrc <idul>@servername.gel.ulaval.ca:/gel/usr/<idul>/.bashrc`

if you ssh with a command (`ssh -c`), it will create a non-interactive shell, which won't load `~/.bashrc`, instead, non-interactive shell will load `~/.profile`. To overcome this and make sure you always get your shell configurations, you can run the following on the remote machine.

`echo "source ~/.bashrc" >> ~/.profile`

Some things to point out here about the server: your `$HOME` directory will always be `/gel/usr/<idul>` and `/home-local/<idul>` (both link point to the same place). There is extra storage available at `/home-local2/<idul>.extra.nobkp/` to put all your data and/or results.

## scp

The basic usage of `scp` is

`scp <SOURCE> <DESTINATION>`


If you want to copy an entire directory, you need to add the `-r` flag like `scp -r <SOURCE> <DESTINATION>`.

Here are some concrete example of `scp`.

##### Copy all files with specific extension in a folder from my computer into the remote machine
`scp /path/to/folder/*.txt <idul>@servername.gel.ulaval.ca:/path/to/folder/`

##### Copy directory and all sub-directories from remote machine into my computer
`scp -r <idul>@servername.gel.ulaval.ca:/path/to/folder /path/to/foler`

##### Copy files between two hosts machine (eg. from __bersimis__ to __mitis__)

The easiest way to do that is ssh in one of the host and then use scp from there.

If you encounter _user unknown_ error while trying to scp between twos hosts, you first need to add the remote host to the known user list (thoses known hosts can be seen in `~/.ssh/known_hosts`). It will be automatically done by trying to `ssh` from within one host to the other. For exemple, start by ssh into __bersimis__, then, from __bersimis__, ssh into __mitis__.

## tmux

[tmux shortcuts&cheatsheet](https://gist.github.com/MohamedAlaa/2961058)

Suppose you want multiple terminals on the host machine, you would need to ssh multiple time. This is not ideal. Further more, if you ssh in, start a training (eg. `python train.py`), then lose internet and then disconnect your ssh, it will kill your terminal on the remote machine and therefore stop your training.

One way to evercome that is to use a terminal multiplexer, eg. `tmux`.

`tmux`is great :D. You can install it on your machine with `sudo apt-get install tmux`. It is already installed on __bersimis__ and __mitis__.

So basically, `tmux` allows you to have multiple terminals in one terminal. To start a tmux session, you can juste type `tmux` in the terminal. But it is preferable to give a name to the session, therefore, you can do the following:

`tmux new -s session-name`

Once you are in your brand new tmux session, you can split vertically or horizontally your terminal into two terminals with `ctrl-b`+`"` and `ctrl-b`+`%`.

If you want a new full size terminal into your tmux session, you can do `ctrl-b`+`c` and use `ctrl-b`+`n` to navigate between them.

To close subterminal, you can do `ctrl-d`

One very nice tool is the option to _detach_ from a tmux session with `ctrl-b`+`d`. Doing so, you will exit you tmux session, but everything in it will keep running as if you were still connected to it.

You can _attach_ back into your tmux session with `tmux a -t session-name`

You can also list all your current tmux sessions with `tmux ls`.

You can do `ctrl-d` to exit a tmux session (this tmux session will no longer be available)

## git/github

git is already installed on __bersimis__ and __mitis__. To setup your account correctly, you just need to add your username and email of your github account.

`git config --global user.name <USERNAME>`

`git config --global user.email <EMAIL>`

## docker & nvidia-docker

On __bersimis__ and __mitis__, it is expected to run everything in a docker container.

I will not go into details on what docker is, instead I will just write out my view of it. There is 4 things:

* __docker__: This is just the name of all that thing, nothing more.

* __dockerfile__: This is a file describing every aspect of a computer on the software level. With this file you know exactly the state of a machine (the environment).

* __image__: An image is the actual computer (or instance) describe in a dockerfile. It is not a description of the computer, it is actually containing every library needed to run it.

* __container__: You might have an image build, but it is doing nothing for the moment, you need _start_ the computer (the image) and give it a specific command to execute. Doing so, you have a container. An image running a command is a container.

Here are more specific explanations for each point.

##### dockerfile
Since it is quite hard to know every libraries needed for a os to run, you will most likely start your _dockerfile_ with the _FROM_ command. This allow you to build _dockerfile_ on top of the other. Here is an example of a _dockerfile_ used for _tensorboard_.

```python
FROM tensorflow/tensorflow

# Export port for TensorBoard
EXPOSE 6006

WORKDIR /tensorboard

# Start running tensorboard when container is launched
ENTRYPOINT ["tensorboard", "--logdir=/tensorboard/"]
```

** Generating _tensorboard_ events is pretty easy with _tensorflow_. To generate _tensorboard_ events with _Pytorch_ you need to install `TensorboardX`.


All my _dockerfile_ are in the same directory on __bersimis__ and __mitis__, for say `/path/to/dockerfiles/tensorboard.docker`

##### image
In order to have an image, you have to build it with

`docker build -f tensorboard.docker --name tensorboard-gab .`

Run this command while being in the folder `/path/to/dockerfiles/`, the `.` is important at the end of the command.

You can list all docker images with `docker images`

##### container
Once the image is build, you can use the command `docker run image-name` to run it. If you want your image to easily have acces to GPU ressources (which is nice when training networks), you should use `nvidia-docker docker run image-name`

You can list all running container with `docker container ls`. If you add the flag `-a` you can see containers that are not running.

`docker run` comes with a lot of options and flags (`docker run --help`). Here are some of the one I use:


`NV_GPU=0 nvidia-docker run --rm -it -v "/path/to/dir:/mnt/dir" --user $(id -u) --shm-size=10g --name container-name image-name`

* __NV_GPU__ isolate the container so a specific GPU.

* __--rm__ Remove the container when it is close. I always use this flag when I run a container.

* __-it__ This actually two flags (__-i__ and __-t__). This one will create a terminal in the container and will make interactive, so you can navigate interactively in the container.

* __-v__ This is very nice, it allows you to mount one of your folder in the container. You can use this option multiple times.

* __--user__ Without this, the container start and put you as root. This is not actually very clever if your container will write file on your computer (like results of a training). I always start container with `--user $(id -u)` which will give me the same permissions as on the remote machine.

* __--shm-size__ If you have memory error while training, try allowing more memory to the container with this option.

* __--name__ Simply gives a name to the container.

* __-w__ Start the container to a specific working directory.

* __-d__ Giving this flag will start the container and _detach_ from it right after.

You can attach to a container with `docker attach container-name`

You can kill container with `docker kill container-name`

#### Useful aliases/functions
Starting container can rapidly get cubersome. Here is two I am using.

`alias doc='nvidia-docker run -it --rm --user $(id -u) --shm-size=10g -v /home-local/galec39.nobkp/path/to/code/:/workspace/code/ -v /home-local2/galec39/path/to/data/:/workspace/data -v /home-local2/galec39/path/to/results:/workspace/results/ -w /workspace/code/'`

With this alias, i can juste do `NV_GPU=0 doc --name container-name image-name` and I start a container.

The dockerfile above is the one I am using for tensorboard only. To run it, I made a function in my `~/.bashrc` file:

```python
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

This function use the image `tensorboard-gab` already build on __bersimis__ and __mitis__, start the container, start tensorboard and detach from it.

So now, suppose that you have _tensorboard_ events in the folder `/path/to/events/`

You can just do `db -p 6006 -v /path/to/events -n container-name`

Simple as that! To close this  container, you have two options: kill it, or attach to it and close it.














