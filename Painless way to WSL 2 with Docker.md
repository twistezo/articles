# Painless way to WSL 2 with Docker

Microsoft introducing Windows Subsystem for Linux (WSL) gave opportunity for developers to run a GNU/Linux environment directly on Windows, unmodified, without the overhead of a virtual machine. This is a great solution because a lot of developers use a different system for programming than for entertainment. WSL is still being developed, but is currently suitable for modern software development. Both in the first and the second version. Looking at the problems and questions reported on the internet, most of them concern the integration of WSL with Docker, especially when it comes to upgrading from WSL 1 to WSL 2. This article will dispel all ambiguities and help you get through this process painlessly. If you don't have WSL 1 and you want to have WSL 2 with Docker you can go straight to the last chapter.

## What will we gain?

- Real latest stable Linux kernel (tuned by Microsoft). WSL 1 has Linux-compatible kernel interface without Linux kernel code.

- Increased file IO performance. Up to 20x faster compared to WSL 1 when unpacking a zipped tarball, and around 2-5x faster when using `git clone`, `npm install` and `cmake` on various projects.

- Full System Call Compatibility. Linux binaries use system calls to perform many functions such as accessing files, requesting memory, creating processes, and more. Whereas WSL 1 used a translation layer that was built by the WSL team, WSL 2 includes its own Linux kernel with full system call compatibility.

- Files are stored in a native ext4 partition on a virtual disk.

- Very easy way to integrate with Docker.

I think for most users file IO performance and easy docker integration will be sufficient reasons.

## Let's go

I assume you have the latest stable version of Windows 10 Pro. For today it is release 1909.

### Join to Windows insiders

WSL 2 currently is only available for Windows 10 in insiders slow ring. As far as I know, it should be available in the stable version in the May 2020 (20H1 release). Slow ring is quite a safe solution for users who want to have new features a little earlier, which are already pre-tested by fast ring users. Slow ring gets system updates once or up to twice a month. You can read about insiders releases changes at Microsoft document called [Flight Hub](https://docs.microsoft.com/en-us/windows-insider/flight-hub/).

For join, follow the official Microsoft [guide](https://insider.windows.com/en-us/for-business-getting-started/#install). It's very easy and takes up about 15 minutes without loosing any personal data.

### Upgrade WSL 1 to WSL 2

Follow official [guide](https://docs.microsoft.com/en-us/windows/wsl/wsl2-install). It is also very easy. A couple of shell commands and here you go. Depending on how extensive your Linux under WSL 1 was, this may take some time. Unfortunately, Microsoft has not placed a progress bar or information about the remaining time so be patient.

### Cleaning up

This is the most important part for proper cooperate between WSL 2 and Docker. Many users after upgrading to WSL 2 are trying to force their version of Docker to cooperate without realizing that Docker especially for WSL 2 has prepared a release that will do everything for us. This is the last release of the Docker Edge version (their beta name) - Docker Desktop WSL 2 backend. This version - 2.1.7.0 is well polished and is a candidate for upcoming stable release.

Users who didn't have WSL 1 or just installed a fresh version of WSL 2 are lucky and can immediately skip to the last chapter.

If you haven't cleaned Docker (at all or correctly) and haven't installed the proper version of Docker for WSL 2 you may have some of the common errors such as `Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?` etc. In most cases, you will lose time not knowing that you have to take a new approach when trying to resolve these errors. But to do this you must first completely remove Docker from WSL and Windows which is not so obvious.

The most common example of unremoved remains is the old Docker approach to set environment variable `DOCKER_HOST=tcp://localhost:2375` under WSL 1 for cooperating with Windows Docker option `Expose daemon on tcp://localhost:2375 without TLS`.

#### How to completely remove Docker from WSL (Ubuntu)?

1. As official Docker [docs](https://docs.docker.com/install/linux/docker-ce/ubuntu/#uninstall-old-versions) says:

   `sudo apt-get remove docker docker-engine docker.io containerd runc`

   For sure you can add to this list also `docker-ce` and `docker-ce-cli`.

2. Identify all docker packages what you have with `dpkg -l | grep -i docker` and remove if any still exists.

3. Remove all residues:

   ```
   sudo rm -rf /var/lib/docker /etc/docker /etc/apparmor.d/docker /var/run/docker.sock /usr/local/bin/docker-compose /etc/docker

   sudo groupdel docker ~/.docker
   ```

4. This step is for advanced users. Be careful. Probably you don't need it.

   Find all `docker` word occurrences and remove these you are sure of.

   `sudo find / -name "*docker*"`

5. Check all below files content for `docker` occurrences like ex. environment variable `DOCKER_HOST=tcp://localhost:2375`.

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

Install latest version of Docker Desktop Edge from [here](https://docs.docker.com/docker-for-windows/edge-release-notes/) and follow steps from official [docs](https://docs.docker.com/docker-for-windows/wsl-tech-preview/). It comes down to a few clicks.

If everything goes fine you should have output from `wsl -l -v` like:

```
  NAME                   STATE           VERSION
* Ubuntu-18.04           Running         2
  docker-desktop         Running         2
  docker-desktop-data    Running         2
```

That's all. Docker has created its own WSL containers `docker-desktop` and `docker-desktop-data`. Some tutorials or advices can be confuse because this time **you shouldn't install or config anything related with Docker under your WSL Linux distrubution**. It's very important. After cleaning it should works out of the box.
