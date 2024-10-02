## Build docker image

```
cd humble
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t l4t-ros2:humble .
```

## Launch docker container

```
xhost +
docker run -it --rm --net=host --runtime nvidia -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix l4t-ros2:humble bash
```
