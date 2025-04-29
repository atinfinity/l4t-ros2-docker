## Build docker image

```bash
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t l4t-ros2:humble .
```

## Launch docker container

```bash
xhost +
docker run -it --rm --net=host --runtime nvidia -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix -v $HOME:$HOME l4t-ros2:humble bash
```
