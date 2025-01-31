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
