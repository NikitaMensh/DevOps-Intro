# Lab 6 Submission

## Task 1 - Container Lifecycle and Image Management

### `docker ps -a`

```sh
CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS                       PORTS     NAMES
ffbe0415ff75   fabook/iros:lidar-lab-v0.0.1   "/ros_entrypoint.sh …"   2 days ago    Exited (137) 2 days ago                lab_lidar-terminal-1
db629fdda920   khronos:jazzy                  "/ros_entrypoint.sh …"   5 days ago    Exited (137) 5 minutes ago             khronos-terminal_linux-1
21908dab1568   kimera-vio-ros:noetic          "/ros_entrypoint.sh …"   6 days ago    Exited (0) 2 days ago                  kimera-vio-ros-terminal_linux-1
00e62099c590   fabook/cv:v0.0.2               "/ros_entrypoint.sh …"   4 weeks ago   Exited (137) 2 weeks ago               lab_0_ros-terminal_linux-1
aba249685d0c   fabook/cv:mr-v0.0.1            "/ros_entrypoint.sh …"   4 weeks ago   Exited (137) 4 weeks ago               mobilerobotics-terminal-1
```

### `docker pull ubuntu:latest`

```sh
latest: Pulling from library/ubuntu
01d7766a2e4a: Already exists
Digest: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

### `docker images ubuntu`

```sh
WARNING: This output is designed for human readability. For machine-readable output, please use --format.
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
ubuntu:latest   bbdabce66f1b       78.1MB             0B        
```

### Inside `ubuntu_container`

`cat /etc/os-release`

```sh
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

`ps aux`

```sh
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.7  0.0   4588  3828 pts/0    Ss   12:04   0:00 /bin/bash
root          10  0.0  0.0   7888  3884 pts/0    R+   12:04   0:00 ps aux
```

### `docker save -o ubuntu_image.tar ubuntu:latest`

This command completed successfully and produced no terminal output.

### `ls -lh ubuntu_image.tar`

```sh
-rw------- 1 nikkimen nikkimen 77M Mar 12 15:04 ubuntu_image.tar
```

### First `docker rmi ubuntu:latest`

```sh
Error response from daemon: conflict: unable to remove repository reference "ubuntu:latest" (must force) - container 5023717d8470 is using its referenced image bbdabce66f1b
```

### `docker rm ubuntu_container`

```sh
ubuntu_container
```

### Second `docker rmi ubuntu:latest`

```sh
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Deleted: sha256:bbdabce66f1b7dde0c081a6b4536d837cd81dd322dd8c99edd68860baf3b2db3
```

### Analysis

- Image size was `78.1MB`.
- Layer count was `1` because `docker image inspect ubuntu:latest` showed one entry in `RootFS.Layers`.
- The exported tar file was `77M`, which is very close to the image size. That is expected because `docker save` writes the image content into a tar archive rather than drastically transforming it.
- Image removal failed because the stopped container still referenced that exact image ID. Docker protects image data while any container depends on it, since the container metadata points to the image layers used as its read-only base filesystem.
- The exported tar file contains the image manifest, image configuration JSON, repository/tag metadata, and the layer tar archive(s) that make up the image filesystem.

## Task 2 - Custom Image Creation and Analysis

### Custom `index.html`

```html
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

### `docker run -d -p 80:80 --name nginx_container nginx`

```sh
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
206356c42440: Pulling fs layer
75a1d70aee50: Pulling fs layer
a9d395129dce: Pulling fs layer
df9da45c1db2: Pulling fs layer
18a071c04bd1: Pulling fs layer
79697674b897: Pulling fs layer
9eef040df109: Pulling fs layer
df9da45c1db2: Waiting
18a071c04bd1: Waiting
79697674b897: Waiting
9eef040df109: Waiting
a9d395129dce: Download complete
df9da45c1db2: Download complete
18a071c04bd1: Verifying Checksum
18a071c04bd1: Download complete
79697674b897: Verifying Checksum
79697674b897: Download complete
9eef040df109: Verifying Checksum
9eef040df109: Download complete
206356c42440: Verifying Checksum
206356c42440: Download complete
206356c42440: Pull complete
75a1d70aee50: Verifying Checksum
75a1d70aee50: Download complete
75a1d70aee50: Pull complete
a9d395129dce: Pull complete
df9da45c1db2: Pull complete
18a071c04bd1: Pull complete
79697674b897: Pull complete
9eef040df109: Pull complete
Digest: sha256:bc45d248c4e1d1709321de61566eb2b64d4f0e32765239d66573666be7f13349
Status: Downloaded newer image for nginx:latest
eb8e6076a00ac9250d1b3c10e7b81812b56c9abc6fdf866858105af5876bf27e
```

### Original `curl http://localhost`

```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   896  100   896    0     0   702k      0 --:--:-- --:--:-- --:--:--  875k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### `docker cp index.html nginx_container:/usr/share/nginx/html/`

This command completed successfully and produced no terminal output.

### Updated `curl http://localhost`

```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    86  100    86    0     0  22933      0 --:--:-- --:--:-- --:--:-- 28666
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

### `docker commit nginx_container my_website:latest`

```sh
sha256:c3b6508e4a361d616a532176839792ba9a28837f5bea7c26b05a74c75c4a86f8
```

### `docker images my_website`

```sh
WARNING: This output is designed for human readability. For machine-readable output, please use --format.
IMAGE               ID             DISK USAGE   CONTENT SIZE   EXTRA
my_website:latest   c3b6508e4a36        161MB             0B        
```

### `docker rm -f nginx_container`

```sh
nginx_container
```

### `docker run -d -p 80:80 --name my_website_container my_website:latest`

```sh
544d69c8f820016f8b6a3a704a165eb16fcb356943885fcc4681803cdd49f4ef
```

### Verification `curl http://localhost`

```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    86  100    86    0     0  46663      0 --:--:-- --:--:-- --:--:-- 86000
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

### `docker diff my_website_container`

```sh
C /run
C /run/nginx.pid
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf
```

### Analysis

- `C` means a file or directory changed relative to the image. In this container, the runtime changed Nginx-related state under `/run` and `/etc`.
- There are no `A` entries because the custom `index.html` was already baked into `my_website:latest` by `docker commit`.
- There are no `D` entries because nothing was deleted from the container filesystem after startup.
- `docker commit` is fast and convenient for capturing an ad hoc container state, which is useful for experiments or debugging.
- `docker commit` is poor for team workflows because it is not declarative, not easily reviewable, and not reproducible.
- A Dockerfile is better for version control, repeatable builds, automation, and long-term maintenance, but it requires explicitly describing every build step.

## Task 3 - Container Networking and Service Discovery

### `docker network create lab_network`

```sh
Error response from daemon: all predefined address pools have been fully subnetted
```

The local Docker environment had exhausted its default bridge address pools, so I created the required network with an explicit subnet:

### `docker network create --subnet 172.30.0.0/24 lab_network`

```sh
4ba47d4809dfab627ce0bed2c8a81e917a37b1ab8472fcb071a386bc91a0343e
```

### `docker network ls`

```sh
NETWORK ID     NAME                      DRIVER    SCOPE
f765aa7bf565   bridge                    bridge    local
6c1e6dab6063   deployment_diabetes_net   bridge    local
ba459d16a00c   host                      host      local
4ba47d4809df   lab_network               bridge    local
3a29d1b55b3b   none                      null      local
e9ae5c1ac404   src_dice                  bridge    local
```

### `docker run -dit --network lab_network --name container1 alpine ash`

```sh
latest: Pulling from library/alpine
589002ba0eae: Pulling fs layer
589002ba0eae: Verifying Checksum
589002ba0eae: Download complete
589002ba0eae: Pull complete
Digest: sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659
Status: Image is up to date for alpine:latest
ea1da4b85cc60a5c3763b7970a79673736234a47b9002418fea9c6c21039b72f
```

### `docker run -dit --network lab_network --name container2 alpine ash`

```sh
latest: Pulling from library/alpine
589002ba0eae: Pulling fs layer
589002ba0eae: Verifying Checksum
589002ba0eae: Download complete
589002ba0eae: Pull complete
Digest: sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659
Status: Downloaded newer image for alpine:latest
30053598b3b72b2a1d2d3c66eb0becb53efa30810a74bdae386696e50093dc0a
```

### `docker exec container1 ping -c 3 container2`

```sh
PING container2 (172.30.0.2): 56 data bytes
64 bytes from 172.30.0.2: seq=0 ttl=64 time=0.071 ms
64 bytes from 172.30.0.2: seq=1 ttl=64 time=0.068 ms
64 bytes from 172.30.0.2: seq=2 ttl=64 time=0.095 ms

--- container2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.068/0.078/0.095 ms
```

### `docker network inspect lab_network`

```sh
[
    {
        "Name": "lab_network",
        "Id": "4ba47d4809dfab627ce0bed2c8a81e917a37b1ab8472fcb071a386bc91a0343e",
        "Created": "2026-03-12T15:11:34.548473515+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.30.0.0/24",
                    "Gateway": "172.30.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {},
        "Labels": {},
        "Containers": {
            "30053598b3b72b2a1d2d3c66eb0becb53efa30810a74bdae386696e50093dc0a": {
                "Name": "container2",
                "EndpointID": "ec9973a6ac3f08ac6df2960affd16a08aac7ac0dd1c2a084a80bd0474934b7ea",
                "MacAddress": "e2:00:53:24:aa:45",
                "IPv4Address": "172.30.0.2/24",
                "IPv6Address": ""
            },
            "ea1da4b85cc60a5c3763b7970a79673736234a47b9002418fea9c6c21039b72f": {
                "Name": "container1",
                "EndpointID": "7f8910cc2567db3cba581f085d3594f33607cfa4f2cc517dac8fb0d3d1211a1c",
                "MacAddress": "e2:cf:cb:41:2f:93",
                "IPv4Address": "172.30.0.3/24",
                "IPv6Address": ""
            }
        },
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.30.0.0/24": {
                        "IPsInUse": 5,
                        "DynamicIPsAvailable": 251
                    }
                }
            }
        }
    }
]
```

### `docker exec container1 nslookup container2`

```sh
Server:		127.0.0.11
Address:	127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:	container2
Address: 172.30.0.2
```

### Analysis

- Docker runs an internal DNS service on user-defined bridge networks. Each container name is registered there, so `container1` can resolve `container2` without manually editing `/etc/hosts`.
- The DNS server address `127.0.0.11` in the `nslookup` output is Docker's embedded resolver inside the container namespace.
- User-defined bridge networks are better than the default `bridge` network because they provide automatic DNS-based name resolution, cleaner isolation, and more predictable multi-container communication.
- The default bridge network usually requires explicit links or manual IP/address management, which is less practical for real services.

## Task 4 - Data Persistence with Volumes

### Custom `index.html`

```html
<html><body><h1>Persistent Data</h1></body></html>
```

### `docker volume create app_data`

```sh
app_data
```

### `docker volume ls`

```sh
DRIVER    VOLUME NAME
local     app_data
local     kolobok_db_data
```

### `docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web nginx`

```sh
316a45f9dc946347822f8db3a8c154a986e21380316b21b95775b27030d96f62
```

### `docker cp index.html web:/usr/share/nginx/html/`

This command completed successfully and produced no terminal output.

### First `curl http://localhost`

```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    51  100    51    0     0  39050      0 --:--:-- --:--:-- --:--:-- 51000
<html><body><h1>Persistent Data</h1></body></html>
```

### `docker stop web && docker rm web`

```sh
web
web
```

### `docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web_new nginx`

```sh
31b9165d09feb32959d43a59e3be516383857a988b30e368f9b1adc9bc3fd9da
```

### Second `curl http://localhost`

```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    51  100    51    0     0  32237      0 --:--:-- --:--:-- --:--:-- 51000
<html><body><h1>Persistent Data</h1></body></html>
```

### `docker volume inspect app_data`

```sh
[
    {
        "CreatedAt": "2026-03-12T15:13:19+03:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/app_data/_data",
        "Name": "app_data",
        "Options": null,
        "Scope": "local"
    }
]
```

### Analysis

- Data persistence matters because containers are disposable. If data stays only in the container writable layer, it disappears when the container is removed.
- Named volumes keep data outside the container lifecycle, which lets a replacement container mount the same data immediately.
- Volumes are the best default for persistent application data managed by Docker, such as databases or uploaded files.
- Bind mounts map a specific host path into a container. They are useful in development when the container must use files directly from the host filesystem.
- Container storage is the writable layer inside a specific container. It is fine for temporary runtime state but not for data that must survive container replacement.
