# aosp-build-docker-images

These are updated versions of the [AOSP Docker files](https://cs.android.com/android/platform/superproject/+/master:build/make/tools/docker/)
which support more modern Linux distribution versions. There's a lot of repetition between them because the 
differences are mainly in the base image and package installation phases.

**Please Note:** Building the AOSP inside docker is convenient, but slow. If possible you should use a bare metal
installation of a Linux distribution to get the best performance.

## Building the images

You can build the docker images using the standard docker build command;

```shell
docker build -f {docker_build_file} -t {your_image_tag} .
```

where `docker_build_file` is the file you want to use to create your build environment,
and `your_image_tag` is a tag you want to give to the final image so it's easy to
remember.

## Running the images

There's a quick and dirty way to do this, and a longer, but more sustainable way; If 
you're testing to see how things work you can use the quick and dirty way to get a 
feel for things, but, if you're planning to do anything more than a quick test, I'd 
advise you to  invest in the more sustainable route.

### Quick and Dirty; Purely inside the docker container

This way is useful if you want to to quickly get started to see how things
work. The problem is you can lose your AOSP checkout and builds very easily 
(e.g. if the docker image gets updated), and each time you lose your checkout
you'll need to download the source again in each one, which can be **very** 
slow.

To start a docker container with the build image you created above you should
run the following command;

```shell
docker run -i -t {your_image_tag}
```

### The sustainable way; Docker Volumes

Docker allows you to use volumes to separate data files from your main image. This
allows you to rebuild your docker image, or even change the entire distribution 
you're building in, without needing to download the AOSP source code again.

First you'll need to create a volume in which you'll store your data by running;

```shell
docker volume create aosp-build
```

then, when you run the image you create above, you'll need to tell docker to mount
the image in a known location. In this example I'll use `/aosp`;

```shell
docker run -i -t --mount source=aosp-build,target=/aosp {your_image_tag}
```

Once the container is running you should do all your work (checkout, build, etc.) in
`/aosp`. If you do anything outside of `/aosp` you risk losing if your docker container
is destroyed, or the image is updated.

## Improving performance on Linux

**WARNING:** This uses your RAM for temporary storage instead of using the disk, if you
don't have enough RAM to support this you'll see strange errors (e.g. unsatisfied link errors,
out of memory errors). If you see this when using this tweak try building without it, or reduce
the number of parallel builds running (i.e. reduce the `-j` value).


If you're using these images on Linux you can make use of [tmpfs](https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html) 
to improve the build performance by adding `--mount type=tmpfs,destination=/app` into your docker run command line.
For the sustanable approach this would give you the following command line;

```shell
docker run -i -t --mount source=aosp-build,target=/aosp --mount type=tmpfs,destination=/tmp {your_image_tag}
```

## Checkout and Build

For instructions on how to check-out and build the AOSP please see the 
[download](https://source.android.com/setup/build/downloading) and
[build](https://source.android.com/setup/build/building) sections of
the AOSP documentation from Google.

## Supporting other Linux distributions

If you have a Dockerfile for another recent distribution please feel free to
create a pull request. I only use Debian these days, so my ability to create 
other images from scratch is limited.