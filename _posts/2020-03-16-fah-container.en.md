---
layout: post
permalink: /posts/2020-03-16-fah-container.html
title: "Running Folding@Home on Linux with CUDA support using Docker"
date: 2020-03-16 20:20:00 -0800
tags: [post, coding, blog]
lang: en
---

Global outbreaks have a way to make you feel powerless. Sitting at home, waiting for the coronavirus to blow over... is there really nothing else you can do? Well, there are ways in which you can help. One such example is [Folding@Home](https://foldingathome.org/). F@H is an initiative that uses your computer's resources to help simulate how proteins and other complex chemicals work. They have set up [high-priority jobs to analyze the 2019-nCoV virus](https://foldingathome.org/2020/02/27/foldinghome-takes-up-the-fight-against-covid-19-2019-ncov/), in hopes to help find antibody targets.

If you want to help, it's as easy as [downloading and running their program](https://foldingathome.org/start-folding/). Well... except for the life of me I couldn't get it to run in Ubuntu. So I decided to find a way to do so.

<!--more-->

## So what was the problem?

I run [Pop!\_OS 19.10](https://system76.com/pop), which is a Linux distro based on Ubuntu. When I tried to install the F@H packages, `apt` told me it can't find `python-gnome2` and some other related dependencies. Apparently, that library is not available from Ubuntu 19.04 onward.

## First attempt: force-install all the dependencies

I found some guides online that try to download and install all the missing dependencies. It's very brute-force-ish: you have to download all the DEB installers yourself, then try to install them and fix dependency issues.

Unsurprisingly, this didn't work. Failed installations, file conflicts... too many issues I didn't care to solve.

## Second attempt: using a Docker container

I talked to some friends in social media about my problem, and somebody came up with the stroke of genius I was missing: use a Docker container! Pre-built containers package all needed dependencies, and there's a good chance somebody has already done so.

Indeed, [there were a few options](https://hub.docker.com/search?q=folding-at-home&type=image). I chose [yurinnick's](https://hub.docker.com/r/yurinnick/folding-at-home) since it was updated recently. This container is lightweight and simple, which are two great pluses.

Unfortunately, something else was missing. While the container ran almost immediately with no fuss, it couldn't detect my graphics card and that also meant no support for CUDA or OpenCL. This is important because GPUs are generally better at massively parallel operations, which is very useful for work loads like analyzing data and folding proteins.

## Third attempt: fixing CUDA and OpenCL support in the container

Luckily for me, my time working as a developer in AWS SageMaker taught me a few things about Docker and working with nVidia containers. Not a lot, to be honest, but good enough to figure out the issue on my own.

Here's how I fixed it:
1. Update your system's GPU drivers.
2. Optional, but potentially useful for debugging: install [nVidia's CUDA toolkit](https://developer.nvidia.com/cuda-downloads). If successful, you should be able to see your current CUDA version when running `nvidia-smi`.
3. Install the [nVidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker). This is crucial because it allows your containers to access your graphics card. You can confirm it worked by running `docker run --gpus all nvidia/cuda:10.0-base nvidia-smi`. If you see the same results as above, it means everything is in order.
4. Finally, I forked [yurinnick's repository](https://github.com/yurinnick/folding-at-home-docker/) and changed the `Dockerfile` to be based on `nvidia/opencl`, which includes all the necessary libraries to allow F@H to use it. My final resulting image is currently [published to Docker Hub and publicly available](https://hub.docker.com/r/thebozzcl/folding-at-home).

After doing the prep steps above and fixing the container image, I was able to run GPU workloads in the container using the following run command:

```
docker run --gpus all \
  --name folding-at-home \
  -p 7396:7396 \
  -p 36330:36330 \
  -e USER=Anonymous \
  -e TEAM=0 \
  -e ENABLE_GPU=true \
  -e ENABLE_SMP=true \
  --restart unless-stopped \
  thebozzcl/folding-at-home \
  --allow 0/0 \
  --web-allow 0/0
```

## Running the container in other OS and hardware configurations

I haven't done a lot of testing, other than in my own machine, so there's not a lot I can say. There's a few things that are pretty certain:
* This should work without much trouble in other machines with nVidia GPUs, as long as you set up the Container Toolkit and keep your drivers up-to-date.
* This should also work in machines without a dedicated GPU, but you should use a lighter image instead.
* I have no idea if this will run with an AMD GPU. I don't know what the process to get them to work looks like... but if you manage to do so, let me know!

## Future steps and pending work... well, not really

I'm afraid I don't want to sustain this container long-term. I'd be willing to fix small emergent issues should they appear and review merge requests, but I have plenty of work in my plate as it is and I don't want to commit long-term to this.

In any case, this container code is dead simple and mostly not mine... so feel free to fork my repo and update the container to your liking!