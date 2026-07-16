Regina in Browser

Screenshot:
![screenshot](screenshot.png)
How to build:

- clone this project
- run `cd tools/docker/regina`
- run `wget https://github.com/regina-normal/regina/releases/download/regina-7.4.1/regina-7.4.1.tar.gz`
- run `./build.sh`
- run `./build-state.js`
- run `cp split.sh ../../../images/split.sh`
- run `cd ../../../images`
- run `./split.sh`
- run `rm debian-9p-rootfs.tar debian-state-base.bin`
- run `cd ..`
- run `make run`

  This should start a server on 8000 (or other ports).

Credit:
- regina: https://regina-normal.github.io/#source
- v86: https://github.com/copy/v86
- sandbox.bio's debian 12 on v86 configuration: https://github.com/sandbox-bio/v86
