# Ascended Container-centric Workstation Setup

Containerising your desktop may seem like it's harder than just installing everything natively, but if you consider the time that goes into getting that perfect setup only to have it broken  by dependency hell a faulty upgrade, or any other source of chaos *when you really need it to work* then suddenly walling all that complexity off into a container seems much simpler, consistent and safer. And it is! This `Readme.md` contains and will accumulate ad-hoc tips and recipes for containerising various apps that some would say are "uncontainerizable", like VPNs and full 3D GPU-accelerated simulation environments. Using a container-centric image-based operating system is strongly recommended, for example Fedora Kinoite, Silverblue, or my personal choice, UBlue Aurora. (Jorge, please dont flame me, I know we're trying to move away from the notion of a "distro", but I have to speak the lingua franca.)


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

## Distrobox - Create a standard ROS2 Dev Environment

```bash
distrobox create humble_env -i osrf/ros:humble-desktop
distrobox enter humble_env
```

## Automatically source ROS script inside Distrobox containers

Have your shell run the following commands at startup, that is add it to, for example: `~/.bashrc.d/ros2.sh`:

```bash
if [[ -n "$CONTAINER_ID" ]]; then
    source /opt/ros/$ROS_DISTRO/setup.bash
fi
```

### How it works

All docker containers have an environment variable called `$CONTAINER_ID`, so if we see that set to any truthy value we know we're in a container. You could add another check to for example, check if the container name starts with `ros`, if you have a consistent naming scheme for your containers. The worst that can happen right now is it tries and fails to source the file in containers without ROS installed.

### Small Note

It's not the best idea to modify `~/.bashrc` directly because system updates may expect a certain set of functions to be there, and there is always a chance your changes could interfere, causing issues from benign and unnoticable to major. They provide a directory for us to place custom aliases and scripts, so please check your shell run commands file for a block that looks for a directory where custom scripts should go. On my system that directory is `~/.bashrc.d/`.

#### Ubuntu
On Ubuntu I had to add add the following block) to `~/.bash_profile` after creating the `~/.bashrc.d/` directory and the `~/.bashrc.d/ros2.sh` file:

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
