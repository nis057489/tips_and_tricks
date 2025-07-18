# Ascended Container-centric Workstation Setup

Containerising your desktop may seem like it's harder than just installing everything natively, but if you consider the time that goes into getting that perfect setup only to have it broken  by dependency hell a faulty upgrade, or any other source of chaos *when you really need it to work* then suddenly walling all that complexity off into a container seems much simpler, consistent and safer. And it is! This `Readme.md` contains and will accumulate ad-hoc tips and recipes for containerising various apps that some would say are "uncontainerizable", like VPNs and full 3D GPU-accelerated simulation environments. Using a container-centric image-based operating system is strongly recommended, for example Fedora Kinoite, Silverblue, or my personal choice, UBlue Aurora. (Jorge, please dont facepalm, I know we're trying to move away from the notion of a "distro", but I have to speak the lingua franca.). For a more general introduction to cloud native OS and the like, check out [Jorge Castro's youtube channel](https://www.youtube.com/watch?v=hn5xNLH-5eA)


# Local VSCode on Remote Container

Use your local VSCode installation to connect to containers running on a remote host.

1. Create a new `docker context`

```bash
docker context create some-context-label --docker "host=ssh://user@remote_server_ip"
```

1. Whenever you use the context, VSCode will list the remote containers instead of your local containers so you can just connect per usual.

```bash
docker context use some-context-label
```

## Convert ROS1 Bag to ROS2 Bag

Install `rosbags-convert`: [rosbags](https://pypi.org/project/rosbags/)

```bash
pip install rosbags
```

Run it on your `.bag` file to convert it into a `.db3` bag. Set the `-dst-typestore` to your ROS version.

```bash
rosbags-convert --src ./your_ros1_bag.bag --dst ./your_ros2_bag --dst-typestore ros2_humble 
```

# Distrobox

Everyone should be using Distrobox! These are some of my curated tips for robotics workflows mainly aimed at an audience that is hearing about Distrobox for the first time. The [official useful tips](https://github.com/89luca89/distrobox/blob/main/docs/useful_tips.md) is so much better though so just have a look, you're sure to find something you'll love!

## Create a standard ROS2 Dev Environment

```bash
distrobox create humble_env -i osrf/ros:humble-desktop-full
distrobox enter humble_env
```

## Clone your perfect Distrobox

Need to simulate a bunch of robots running the same ROS distro? Clone an existing Distrobox with `--clone`

```bash
distrobox create humble_env -i osrf/ros:humble-desktop-full
distrobox create --name cloned_humble_env --clone humble_env

# Terminal 1
distrobox enter humble_env
# Terminal 2
distrobox enter cloned_humble_env
```

## Broken container? Delete it and recreate!

Because your files are stored in your host machine, not the container, you can delete and recreate the container without losing your files.

```shell
distrobox rm that_broken_env
```

## The `-p` flag

```bash
distrobox create jazzy_env -i -p osrf/ros:jazzy-desktop-full
distrobox enter jazzy_env
```

`--pull` or `-p` for short, will pull the latest image. Handy ~~if~~*when* you encounter pesky repository *key expriation errors* like the one shown below. 

```shell
W: An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: http://packages.ros.org/ros2/ubuntu noble InRelease: The following signatures were invalid: EXPKEYSIG F42ED6FBAB17C654 Open Robotics <info@osrfoundation.org>
```

But don't let chasing up stuff like this threaten your deadline, you already know how to `rm` and recreate the container from the latest image with the `-p` flag!

## Using the GPU

Is your host machine running NVidia proprietary drivers and do want to use the GPU in the containers? Add the `--nvidia` flag during creation. Running AMD or Intel? Support is baked in, [you don't need to do anything else](https://github.com/89luca89/distrobox/blob/main/docs/useful_tips.md#using-the-gpu-inside-the-container).

```bash
distrobox create --nvidia kilted_env -i -p  osrf/ros:kilted-desktop-full
```

Note: the `--nvidia` flag [doesn't work on ancient operating systems](https://github.com/89luca89/distrobox/blob/main/docs/usage/distrobox-create.md#nvidia-integration), those would need to [use the conatiner-toolkit](https://github.com/89luca89/distrobox/blob/main/docs/usage/distrobox-create.md#nvidia-integration).


## Automatically source ROS script inside Distrobox containers

Many people dont want to have to type `source /opt/ros/humble/setup.bash` every time they open a shell in a container; that seems like something the computer should do for us.

To make it so, create a file somewhere, for example: `~/.bashrc.d/ros2.sh`, with the following contents:

```bash
if [[ -n "$CONTAINER_ID" ]]; then
    source /opt/ros/$ROS_DISTRO/setup.bash
fi
```

Here is a pastable one-liner to do that for you:

```bash
mkdir -p ~/.bashrc.d && cat > ~/.bashrc.d/ros2.sh <<'EOF'
if [[ -n "$CONTAINER_ID" ]]; then
    source /opt/ros/$ROS_DISTRO/setup.bash
```

Then edit your `~/.bash_profile` file and add this block if it does not exist:

```bash
# User specific aliases and functions
if [ -d ~/.bashrc.d ]; then
    for rc in ~/.bashrc.d/*; do
        if [ -f "$rc" ]; then
            . "$rc"
        fi
    done
fi
unset rc
```

### How it works

All docker containers have an environment variable called `$CONTAINER_ID`, so if we see that set to any truthy value we know we're in a container. You could add another check to for example, check if the container name starts with `ros`, if you have a consistent naming scheme for your containers. The worst that can happen right now is it tries and fails to source the file in containers without ROS installed.

### Small Note

It's not the best idea to modify `~/.bashrc` directly because system updates may expect a certain set of functions to be there, and there is always a chance your changes could interfere, causing issues from benign and unnoticable to major. They provide a directory for us to place custom aliases and scripts, so please check your shell run commands file for a block that looks for a directory where custom scripts should go. On my system that directory is `~/.bashrc.d/`.



### Distrobox F5 VPN Setup
Tl;dr Install the RPM version of F5 into a Fedora 42 contianer using distrobox. Exported the f5 app, launch the VPN by clicking the link in the F5 VPN website. 

For reference, the container must be rootful and have net admin permissions and be using systemd to work with the F5 daemon.

#### Commands

##### Distrobox creation command

```bash
distrobox create utsvpn --init --image fedora:42 -a "--cap-add=NET_ADMIN" --additional-packages systemd --root
```

##### Install the F5 GUI client

```bash
sudo dnf install -y ~/Downloads/linux_f5vpn.x86_64_v7251.2025.rpm
```

##### Export the F5 app to my host so it behaves like a locally-installed app (launchable by the browser launcher and appearing in the apps menu)

```bash
distrobox-export --app /opt/f5/vpn/com.f5.f5vpn.desktop
```

#### Launch

Tl;dr Quickly launch the F5 client using this one liner then start the VPN on the website. You may be prompted to enter your machine's root password to complete launching the of the app.

```bash
xhost +local:root && distrobox enter --root utsvpn -- /opt/f5/vpn/f5vpn %u && exit
```

##### Explaination

The command will allow GUI apps to run as root, enter your container (here, called utsvpn) start the F5 client then exit. Note, the `utsvpn` container here is based on a Fedora image. If using a Debian image your path may vary, but you can get it by checking the Exec line, shown in the Troubleshooting section.

#### Troubleshooting

If you have any issues you can check the installation is correct with

```bash
rpm -ql f5vpn | grep -E 'bin/|.desktop'

# should output: 
# /opt/f5/vpn/com.f5.f5vpn.desktop
# /opt/f5/vpn/xdg-scripts/xdg-desktop-menu

# to see the actual launch command run:
grep Exec /opt/f5/vpn/com.f5.f5vpn.desktop
```

## Nvidia Isaac SIM inside Distrobox

Best to refer to my [repo](https://github.com/nis057489/isaac_sim_wayland)
