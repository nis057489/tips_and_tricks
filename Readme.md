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

Install `rosbags-convert`:

```bash
pip install rosbag
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

For example my `.bashrc` contains this block:

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
