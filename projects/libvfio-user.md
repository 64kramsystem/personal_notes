# libvfio-user

- [libvfio-user](#libvfio-user)
  - [Setup/Build/Jobs](#setupbuildjobs)
  - [General info](#general-info)

## Setup/Build/Jobs

```sh
apt install libjson-c-dev libcmocka-dev libssl-dev python3-pytest

# Produces libvfio-user.so
make BUILD_TYPE=rel
make install

make coverity   # upload a coverity build
make pytest
```

## General info

- API info: `include/libvfio-user.h`
