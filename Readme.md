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
