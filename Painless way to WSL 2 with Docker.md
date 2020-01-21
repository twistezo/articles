# Painless way to WSL 2 with Docker

By introducing the Windows Subsystem for Linux (WSL), Microsoft gave the opportunity for developers to run a GNU/Linux environment directly on Windows, unmodified, without the overhead of a virtual machine. This is a great solution because a lot of developers use one system for programming and another for entertainment.

WSL is still being developed, but it is currently suitable for modern software development, both in the first and the second version. Browsing through the problems and questions reported on the internet, most of them seem to concern the integration of WSL with Docker, especially when it comes to upgrading from WSL 1 to WSL 2. This article will dispel all ambiguities and help you get through this process painlessly. If you don’t have WSL 1 and you want to have WSL 2 with Docker, you can jump straight to the last chapter.

## What will you gain?

- Real latest stable Linux kernel (tuned by Microsoft). WSL 1 has a Linux-compatible kernel interface without the Linux kernel code.

- Increased file IO performance. Up to 20x faster compared to WSL 1 when unpacking a zipped tarball, around 2-5x faster when using `git clone`, `npm install` and `cmake` on various projects.

- Full System Call Compatibility. Linux binaries use system calls to perform many functions, such as accessing files, requesting memory, creating processes, and many more. While WSL 1 used a translation layer built by the WSL team, WSL 2 includes its own Linux kernel with full system call compatibility.

- Files stored in a native ext4 partition on a virtual disk.

- Very easy way to integrate with Docker.

I think for most users the file IO performance and easy docker integration sound like sufficient reasons.

## Let's go

I assume you have the latest stable version of Windows 10 Pro. As for today, it is the 1909 release.

### Join Windows insiders

Currently, WSL 2 is only available for Windows 10 in insiders’ slow ring. As far as I know, the stable version should be available in May 2020 (20H1 release). The slow ring is quite a safe solution for users who want to have new features a little earlier, already pre-tested by fast ring users. The slow ring gets system updates up to once or twice a month. You can read about insiders release changes in the Microsoft document called [Flight Hub](https://docs.microsoft.com/en-us/windows-insider/flight-hub/).

To join, simply follow the official Microsoft [guide](https://insider.windows.com/en-us/for-business-getting-started/#install). It’s very easy and takes up about 15 minutes without losing any personal data.

### Upgrade WSL 1 to WSL 2

Follow the official [guide](https://docs.microsoft.com/en-us/windows/wsl/wsl2-install). It is also very easy, a couple of shell commands and here you are! Depending on how extensive your Linux under WSL 1 was, this may take some time. Unfortunately, Microsoft has not placed a progress bar or information about the required time, so just be patient.

### Cleaning up

This is the most important part for proper cooperation of WSL 2 and Docker. After upgrading to WSL 2, many users are trying to force their version of Docker to cooperate without realizing that Docker team, especially for WSL 2, has prepared a release that will do everything for us. This is the last release of the Docker Edge version (their beta name) – “Docker Desktop WSL 2 backend”. This version, 2.1.7.0, is a well-polished and is a candidate for the upcoming stable release.

Users who didn’t have WSL 1 or just installed a fresh version of WSL 2 are lucky and can immediately skip to the last chapter.

If you haven’t cleaned your Docker (at all or correctly) and haven’t installed the proper version of Docker for WSL 2, you may encounter some common errors, e.g., `Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?`, etc. In most cases, you will lose time if you do not know that you must take a new approach in trying to resolve these errors. To do this, you must first completely remove Docker from WSL and Windows, which is not an obvious step.

The most common example of unremoved remains is the old Docker approach to set the environment variable `DOCKER_HOST=tcp://localhost:2375` under WSL 1 for cooperating with Windows Docker option `Expose daemon on tcp://localhost:2375 without TLS`.

#### How to completely remove Docker from WSL (Ubuntu)?

1. As the official Docker [docs](https://docs.docker.com/install/linux/docker-ce/ubuntu/#uninstall-old-versions) says:

   `sudo apt-get remove docker docker-engine docker.io containerd runc`

   For sure, you can add to this list `docker-ce` and `docker-ce-cli`.

2. Identify all docker packages that you have with `dpkg -l | grep -i docker` and remove if any still exists.

3. Remove all residues:

   ```
   sudo rm -rf /var/lib/docker /etc/docker /etc/apparmor.d/docker /var/run/docker.sock /usr/local/bin/docker-compose /etc/docker

   sudo groupdel docker ~/.docker
   ```

4. This step is for advanced users, so be careful. Probably you don't need to make it.

   Find all `docker` word occurrences and remove the ones you are sure of.

   `sudo find / -name "*docker*"`

5. Check the content of all files below for `docker` occurrences like ex. environment variable `DOCKER_HOST=tcp://localhost:2375`.

   You can edit these files with the Nano editor `nano ~/.bashrc` or Visual Studio Code `code ~/.bashrc`.

   ```
   ~/.bashrc
   ~/.bash_aliases
   ~/.bash_profile
   ~/.bash_login
   ~/.profile
   /etc/bash.bashrc
   /etc/profile
   ```

#### How to completely remove Docker from Windows?

1. Uninstall in normal way with Windows `Apps & features` panel.
2. Remove all below if exists:

   ```
   C:\Program Files\Docker
   C:\ProgramData\DockerDesktop
   C:\Users\[USERNAME]\.docker
   C:\Users\[USERNAME]\AppData\Local\Docker
   C:\Users\[USERNAME]\AppData\Roaming\Docker
   C:\Users\[USERNAME]\AppData\Roaming\Docker Desktop
   ```

### What next?

Install the latest version of Docker Desktop Edge from [here](https://docs.docker.com/docker-for-windows/edge-release-notes/) and follow steps from official [docs](https://docs.docker.com/docker-for-windows/wsl-tech-preview/). It really comes down to a few clicks.

If everything goes well, you should have your output from `wsl -l -v` like this:

```
  NAME                   STATE           VERSION
* Ubuntu-18.04           Running         2
  docker-desktop         Running         2
  docker-desktop-data    Running         2
```

That's all. Docker has created its own WSL containers `docker-desktop` and `docker-desktop-data`. Some of the available tutorials or advices can be confusing because this time **you shouldn't install or config anything related to Docker under your WSL Linux distrubution**. It’s very important to remember. After the cleaning, it should work well right away.
