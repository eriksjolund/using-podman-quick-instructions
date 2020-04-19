# Using [podman](https://podman.io/) quick instructions

These instructions assume that

* you have some knowledge about basic Linux commands such as __ls__, __cat__, __echo__ and __grep__
* you have a Linux computer with __podman__ installed. ([Installation instructions](https://podman.io/getting-started/installation))

#### Table of contents

- [Basic usage](#basic-usage)
  * [The difference between the host system and the container](#the-difference-between-the-host-system-and-the-container)
  * [Files on the host system are not accessible from within the container by default](#files-on-the-host-system-are-not-accessible-from-within-the-container-by-default)
  * [How to let the container read and write to a directory on the host system](#how-to-let-the-container-read-and-write-to-a-directory-on-the-host-system)
  * [Popular Linux container images](#popular-Linux-container-images)
- [Install a missing software package](#install-a-missing-software-package)
  * [Is the software package already installed?](#is-the-software-package-already-installed)
    + [Example: Fedora](#example-fedora--is-graphicsmagick-already-installed)
    + [Example: CentOS](#example-centos--is-graphicsmagick-already-installed)
    + [Example: Ubuntu LTS](#example-ubuntu-lts--is-graphicsmagick-already-installed)
    + [Example: Alpine](#example-alpine--is-graphicsmagick-already-installed)
    + [Results of the tests](#results-of-the-tests-is-graphicsmagick-already-installed)
  * [Is the software package available in a package repository?](#is-the-software-package-available-in-a-package-repository)
    + [Example: Fedora](#example-fedora--is-graphicsmagick-available-in-a-package-repository)
    + [Example: CentOS](#example-centos--is-graphicsmagick-available-in-a-package-repository)
    + [Example: Ubuntu LTS](#example-ubuntu-lts--is-graphicsmagick-available-in-a-package-repository)
    + [Example: Alpine](#example-alpine--is-graphicsmagick-available-in-a-package-repository)
    + [Results of the tests](#results-of-the-tests-is-graphicsmagick-available-in-a-package-repository)
  * [Use _podman build_ to run an install command and then save the result to a new container image](#use-podman-build-to-run-an-install-command-and-then-save-the-result-to-a-new-container-image)
    + [Example: Fedora](#example-fedora--install-graphicsmagick-with-podman-build)
    + [Example: CentOS](#example-centos--install-graphicsmagick-with-podman-build)
    + [Example: Ubuntu LTS](#example-ubuntu-lts--install-graphicsmagick-with-podman-build)
    + [Example: Alpine](#example-alpine--install-graphicsmagick-with-podman-build)
    + [Compare the sizes of the built images](#compare-the-sizes-of-the-built-images)
  * [Use the installed software package (resize a photo with GraphicsMagick)](#use-the-installed-software-package-resize-a-photo-with-graphicsmagick)
    + [Example: Fedora](#example-fedora--resize-a-photo-with-graphicsmagick)
    + [Example: CentOS](#example-centos--resize-a-photo-with-graphicsmagick)
    + [Example: Ubuntu LTS](#example-ubuntu-lts--resize-a-photo-with-graphicsmagick)
    + [Example: Alpine](#example-alpine--resize-a-photo-with-graphicsmagick)
- [Search for a pre-built container image](#search-for-a-pre-built-container-image)
  * [Example: Search docker.io](#example-search-dockerio)
- [Security](#security)
  * [Consider the risc of malicious code in pre-built container images](#consider-the-risc-of-malicious-code-in-pre-built-container-images)
  * [How to run a command in a container image in a more secure and restricted way](#how-to-run-a-command-in-a-container-image-in-a-more-secure-and-restricted-way)
- [How to save disk space](#how-to-save-disk-space)
- [How to save time](#how-to-save-time)
- [When to use the flags _-i_ (_--interactive_) and _-t_ (_--tty_)](#when-to-use-the-flags--i---interactive-and--t---tty)
- [The professional way, using Dockerfile and Github/Gitlab](#the-professional-way-using-dockerfile-and-githubgitlab)

## Basic usage

Show the podman version

 ```
[me@linux ~]$ podman --version
podman version 1.8.0
[me@linux ~]$ 
```

Run the command `echo Hello!` in the container __docker.io/library/ubuntu:18.04__

```
[me@linux ~]$ podman run --rm docker.io/library/ubuntu:18.04 echo Hello!
Trying to pull docker.io/library/ubuntu:18.04...
Getting image source signatures
Copying blob de83a2304fa1 done  
Copying blob b6b53be908de done  
Copying blob 423ae2b273f4 done  
Copying blob f9a83bce3af0 done  
Copying config 72300a873c done  
Writing manifest to image destination
Storing signatures
Hello!
[me@linux ~]$ 
```

The first lines of the text were printed to _stderr_ telling us that
podman needed to download the container image.

The last line of the text was printed to _stdout_  and originates from the command `echo Hello!` that was run inside the container.

If we run the same command again 

```
[me@linux ~]$ podman run --rm docker.io/library/ubuntu:18.04 echo Hello!
Hello!
[me@linux ~]$ 
```

we see that the download step could be skipped.

The size of the downloaded container image __docker.io/library/ubuntu:18.04__ is 66.6 MB
```
[me@linux ~]$ podman images
REPOSITORY                  TAG           IMAGE ID       CREATED        SIZE
docker.io/library/ubuntu    18.04         72300a873c2c   4 days ago     66.6 MB
[me@linux ~]$ 
```

but container images can be much smaller than that. For instance the popular container image 
__docker.io/library/alpine:latest__ is about 6 MB. 


## The difference between the host system and the container

If we run the command  `cat /etc/os-release` inside the container

```
[me@linux ~]$ podman run -ti --rm docker.io/library/ubuntu:18.04 cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
[me@linux ~]$ 
```
we see the contents of the file _/etc/os-release_ inside the container image.

If we run `cat /etc/os-release` without podman

```
[me@linux ~]$ cat /etc/os-release
NAME=Fedora
VERSION="31 (Thirty One)"
ID=fedora
VERSION_ID=31
VERSION_CODENAME=""
PLATFORM_ID="platform:f31"
PRETTY_NAME="Fedora 31 (Thirty One)"
ANSI_COLOR="0;34"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:31"
HOME_URL="https://fedoraproject.org/"
DOCUMENTATION_URL="https://docs.fedoraproject.org/en-US/fedora/f31/system-administrators-guide/"
SUPPORT_URL="https://fedoraproject.org/wiki/Communicating_and_getting_help"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Fedora"
REDHAT_BUGZILLA_PRODUCT_VERSION=31
REDHAT_SUPPORT_PRODUCT="Fedora"
REDHAT_SUPPORT_PRODUCT_VERSION=31
PRIVACY_POLICY_URL="https://fedoraproject.org/wiki/Legal:PrivacyPolicy"
[me@linux ~]$ 
```
we see the contents of the file _/etc/os-release_ on the host system. 

## Files on the host system are not accessible from within the container by default

The home directory

```
[me@linux ~]$ ls -d /home/me
/home/me
[me@linux ~]$ 
```

on the host system is not available from inside the container

```
[me@linux ~]$ podman run -ti --rm docker.io/library/ubuntu:18.04 ls -d /home/me
ls: cannot access '/home/me': No such file or directory
[me@linux ~]$ 
```

This is because the host system and the container have different file trees.

## How to let the container read and write to a directory on the host system

To run a command in the container with read and write access to the directory _/home/me/project1_ on the host system,
add the command line
option `-v /home/me/project1:/some/path:Z`.

```
[me@linux ~]$ echo abc > /home/me/project1/file1.txt
[me@linux ~]$ ls /home/me/project1
file1.txt
[me@linux ~]$ cat /home/me/project1
abc
[me@linux ~]$ podman run -ti --rm docker.io/library/ubuntu:18.04 -v /home/me/project1:/some/path:Z cat /some/path/file1.txt
abc
[me@linux ~]$ podman run -ti --rm docker.io/library/ubuntu:18.04 -v /home/me/project1:/some/path:Z rm /some/path/file1.txt
[me@linux ~]$ ls /home/me/project1
[me@linux ~]$ 
```

The path inside the container _/some/path_ was chosen arbitrarily.
To simplify things, you would often want to use the same path both outside and inside the container, i.e using the command line
option `-v /home/me/project1:/home/me/project1:Z`.

## Popular Linux container images

| Container image | Size (MB)   | Comment |
| ---- | --      | --      |
| registry.fedoraproject.org/fedora:31 | 200 | Fedora 31 |
| docker.io/library/fedora:31 | 200 | docker.io also provides Fedora 31 |
| docker.io/library/centos:8 | 245 | CentOS 8 | 
| docker.io/library/ubuntu:19.10 | 75 | Ubuntu 19.10 | 
| docker.io/library/ubuntu:18.04 | 67 | Ubuntu 18.04 LTS (Long Term Support) |
| docker.io/library/alpine:3 | 6 | If you want to create very small container images. :warning: Requires a bit more expertise to work with. |

# Install a missing software package

__Goal of the exercise__: 

Resize the photo _~/img/photo.jpg_ 

![](images/photo.jpg)

to half the size

![](images/resized_photo.jpg)

and save it to _~/img/resized_photo.jpg_ 
by using `gm convert -resize 50%` (from the software package [__GraphicsMagick__](http://www.graphicsmagick.org)).

## Is the software package already installed?

Is GraphicsMagick already installed?

(`-ti` is not needed when `podman run` is used in a pipe)

#### Example Ubuntu LTS : Is GraphicsMagick already installed?

Check the container image __docker.io/library/ubuntu:18.04__

```
[me@linux ~]$ podman run --rm docker.io/library/ubuntu:18.04 dpkg -l | grep -i graphicsmagick
[me@linux ~]$ 
```

#### Example Fedora : Is GraphicsMagick already installed?

Check the container image __registry.fedoraproject.org/fedora:31__
```
[me@linux ~]$ podman run --rm registry.fedoraproject.org/fedora:31 rpm -qa | grep -i graphicsmagick
[me@linux ~]$ 
```
#### Example CentOS : Is GraphicsMagick already installed?


Check the container image __docker.io/library/centos:8__
```
[me@linux ~]$ podman run --rm docker.io/library/centos:8 rpm -qa | grep -i graphicsmagick
[me@linux ~]$ 
```

#### Example Alpine : Is GraphicsMagick already installed?

Check the container image __docker.io/library/alpine:3__

```
[me@linux ~]$ podman run --rm docker.io/library/alpine:3 apk --no-cache list -a | grep -i graphicsmagick
[me@linux ~]$ 
```

#### Results of the tests: Is GraphicsMagick already installed?

| Container image | GraphicsMagick already installed? (Yes/No) |
| ---- | --      |
| docker.io/library/ubuntu:18.04  | No | 
| registry.fedoraproject.org/fedora:31 | No |
| docker.io/library/centos:8 | No |
| docker.io/library/alpine:3 | No | 


## Is the software package available in a package repository?

Is GraphicsMagick available in a package repository?

#### Example Fedora : Is GraphicsMagick available in a package repository?

Check the container image __registry.fedoraproject.org/fedora:31__
```
[me@linux ~]$ podman run --rm registry.fedoraproject.org/fedora:31 dnf list available | grep -i graphicsmagick
GraphicsMagick.i686                                                        1.3.34-1.fc31                                                    updates        
GraphicsMagick.x86_64                                                      1.3.34-1.fc31                                                    updates        
GraphicsMagick-c++.i686                                                    1.3.34-1.fc31                                                    updates        
GraphicsMagick-c++.x86_64                                                  1.3.34-1.fc31                                                    updates        
GraphicsMagick-c++-devel.i686                                              1.3.34-1.fc31                                                    updates        
GraphicsMagick-c++-devel.x86_64                                            1.3.34-1.fc31                                                    updates        
GraphicsMagick-devel.i686                                                  1.3.34-1.fc31                                                    updates        
GraphicsMagick-devel.x86_64                                                1.3.34-1.fc31                                                    updates        
GraphicsMagick-doc.noarch                                                  1.3.34-1.fc31                                                    updates        
GraphicsMagick-perl.x86_64                                                 1.3.34-1.fc31                                                    updates    
[me@linux ~]$ 
```

(The website 
https://apps.fedoraproject.org/packages/
provides the same information)

#### Example CentOS : Is GraphicsMagick available in a package repository?

Check the container image __docker.io/library/centos:8__
```
[me@linux ~]$ podman run --rm docker.io/library/centos:8 dnf list available | grep -i graphicsmagick
[me@linux ~]$ 
```

Nothing there.

Let's check the extra repository PowerTools

```
[me@linux ~]$ podman run --rm docker.io/library/centos:8 dnf repository-packages PowerTools list | grep -i graphicsmagick
[me@linux ~]$ 
```
Nothing there.

Let's check the third-party repository [EPEL](https://fedoraproject.org/wiki/EPEL).

```
[me@linux ~]$ podman run --rm docker.io/library/centos:8 /bin/bash -c "dnf -y update && dnf -y install epel-release && dnf repository-packages epel list" | grep -i graphicsmagick
warning: /var/cache/dnf/AppStream-02e86d1c976ab532/packages/libxkbcommon-0.8.2-1.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
GraphicsMagick.x86_64                              1.3.34-1.el8                           epel
GraphicsMagick-c++.x86_64                          1.3.34-1.el8                           epel
GraphicsMagick-c++-devel.x86_64                    1.3.34-1.el8                           epel
GraphicsMagick-devel.x86_64                        1.3.34-1.el8                           epel
GraphicsMagick-doc.noarch                          1.3.34-1.el8                           epel
GraphicsMagick-perl.x86_64                         1.3.34-1.el8                           epel
[me@linux ~]$ 
```


#### Example Ubuntu LTS : Is GraphicsMagick available in a package repository?

Check the container image __docker.io/library/ubuntu:18.04__

```
[me@linux ~]$ podman run --rm docker.io/library/ubuntu:18.04 /bin/bash -c "apt-get update && apt-cache search GraphicsMagick" | grep -i ^graphicsmagick
graphicsmagick - collection of image processing tools
graphicsmagick-dbg - format-independent image processing - debugging symbols
graphicsmagick-imagemagick-compat - image processing tools providing ImageMagick interface
graphicsmagick-libmagick-dev-compat - image processing libraries providing ImageMagick interface
[me@linux ~]$ 
```

(The website 
https://packages.ubuntu.com/
provides the same information)

#### Example Alpine : Is GraphicsMagick available in a package repository?

Check the container image  __docker.io/library/alpine:3__

```
[me@linux ~]$ podman run --rm docker.io/library/alpine:3 apk --no-cache list -a | grep -i graphicsmagick
graphicsmagick-zsh-completion-5.7.1-r0 x86_64 {zsh} (custom)
graphicsmagick-1.3.33-r1 x86_64 {graphicsmagick} (MIT)
graphicsmagick-dev-1.3.33-r1 x86_64 {graphicsmagick} (MIT)
graphicsmagick-doc-1.3.33-r1 x86_64 {graphicsmagick} (MIT)
[me@linux ~]$ 
```

(The website
https://pkgs.alpinelinux.org/packages
provides the same information)


#### Results of the tests: Is GraphicsMagick available in a package repository?

 GraphicsMagick is available in package repositories!


| Container image | Command to install GraphicsMagick |
| ---- | --      |
| docker.io/library/ubuntu:18.04  | apt-get update && apt-get install -y graphicsmagick | 
| registry.fedoraproject.org/fedora:31 | dnf -y update && dnf -y install GraphicsMagick |
| docker.io/library/centos:8 | dnf -y update && dnf -y install epel-release && dnf -y install GraphicsMagick |
| docker.io/library/alpine:3 | apk --no-cache add graphicsmagick | 


But you will need to use `podman build` to save the result to a new container image.

## Use _podman build_ to run an install command and then save the result to a new container image


#### Example Fedora : Install GraphicsMagick with _podman build_

To install GraphicsMagick on `docker.io/library/fedora:31` and save the result to the new container image
`localhost/foobar:fedora31`, copy-paste these lines into the terminal

```
echo "FROM docker.io/library/fedora:31
RUN dnf -y update && dnf -y install GraphicsMagick && dnf clean all
" | podman build -t foobar:fedora31 -

``` 

Show details about the new container 

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:fedora31
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   fedora31   3ea792460e93   20 seconds ago   259 MB
[me@linux ~]$ 
```

The ` && dnf clean all` was added to reduce the size of the container image, otherwise the size would have been 478 MB.

The text strings _foobar_ and _fedora31_ in `localhost/foobar:fedora31` were arbitrarily chosen. The syntax is _localhost/name:tag_.

#### Example CentOS : Install GraphicsMagick with _podman build_

To install GraphicsMagick on `docker.io/library/centos:8` and save the result to the new container image
`localhost/foobar:centos8`, copy-paste these lines into the terminal


```
echo "FROM docker.io/library/centos:8
RUN dnf -y update && dnf -y install epel-release && dnf -y install GraphicsMagick && dnf clean all
" | podman build -t foobar:centos8 -

``` 

Show details about the new container

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:centos8
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   centos8   ddbad21e0684   36 seconds ago   395 MB
[me@linux ~]$ 
```

The ` && dnf clean all` was added to reduce the size of the container image, otherwise the size would have been 433 MB.

The text strings _foobar_ and _centos8_ in `localhost/foobar:centos8` were arbitrarily chosen. The syntax is _localhost/name:tag_.



#### Example Ubuntu LTS : Install GraphicsMagick with _podman build_

To install GraphicsMagick on `docker.io/library/ubuntu:18.04` and save the result to the new container image
`localhost/foobar:ubuntu1804`, copy-paste these lines into the terminal

```
echo "FROM docker.io/library/ubuntu:18.04
RUN apt-get update && apt-get install -y --no-install-recommends graphicsmagick && rm -rf /var/lib/apt/lists/*
" | podman build -t foobar:ubuntu1804 -

```

Show details about the new container

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:ubuntu1804
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   ubuntu1804   891d0886ab51  15 seconds ago   112 MB
[me@linux ~]$ 
```

Both `--no-install-recommends` and  ` && rm -rf /var/lib/apt/lists/*` were added to reduce the size of the container image. 

| Image size | Installation command |
| ----       | --                   |
| 198 MB       | apt-get update && apt-get install -y graphicsmagick | 
| 170 MB       | apt-get update && apt-get install -y graphicsmagick && rm -rf /var/lib/apt/lists/* | 
| 112 MB      | apt-get update && apt-get install -y --no-install-recommends graphicsmagick && rm -rf /var/lib/apt/lists/* | 

The text strings _foobar_ and _ubuntu1804_ in `localhost/foobar:ubuntu1804` were arbitrarily chosen. The syntax is _localhost/name:tag_.


#### Example Alpine : Install GraphicsMagick with _podman build_

To install GraphicsMagick on `docker.io/library/alpine:3` and save the result to the new container image
`localhost/foobar:alpine3`, copy-paste these lines into the terminal

```
echo "FROM docker.io/library/alpine:3
RUN apk --no-cache add graphicsmagick
" | podman build -t foobar:alpine3 -

``` 

Show details about the new container

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:alpine3
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
localhost/foobar   alpine3   93a5ac124e03   35 seconds ago   24.8 MB
[me@linux ~]$ 
``` 



The text strings _foobar_ and _alpine3_ in `localhost/foobar:alpine3` were arbitrarily chosen. The syntax is _localhost/name:tag_.


#### Compare the sizes of the built images

The container images ordered in size

| Container image  | Installation command |
| ----             | --                   |
| localhost/foobar:alpine3 | 24.8 MB |
| localhost/foobar:ubuntu1804 | 112 MB |
| localhost/foobar:fedora31 | 259 MB |
| localhost/foobar:centos8 | 395 MB |

## Use the installed software package (resize a photo with GraphicsMagick)

Although the containers have been built with different commands and from different Linux distributions,
the resulting container images will behave quite similarly when they are used.

The following examples show how similar they are from a user perspective.

#### Example Fedora : Resize a photo with GraphicsMagick

Let's use the [previously built](#example-fedora--install-graphicsmagick-with-podman-build) container image __localhost/foobar:fedora31__

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:fedora31
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   fedora31   3ea792460e93   1 hour ago   259 MB
[me@linux ~]$ 
```

to resize the photo _/home/me/img/photo.jpg_

```
[me@linux ~]$ ls /home/me/img/
photo.jpg
[me@linux ~]$ podman run -ti --rm -v /home/me/img:/home/me/img:Z localhost/foobar:fedora31 gm convert -resize 50% /home/me/img/photo.jpg /home/me/img/resized_photo.jpg
[me@linux ~]$ ls /home/me/img/
photo.jpg
photo_resized.jpg
[me@linux ~]$ 
``` 

Show the GraphicsMagick version

``` 
[me@linux ~]$ podman run --rm localhost/foobar:fedora31 gm version | head -1
GraphicsMagick 1.3.34 2019-12-24 Q16 http://www.GraphicsMagick.org/
``` 

#### Example CentOS : Resize a photo with GraphicsMagick

Let's use the [previously built](#example-centos--install-graphicsmagick-with-podman-build) container image __localhost/foobar:centos8__

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:centos8
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   centos8   ddbad21e0684   1 hour ago   395 MB
[me@linux ~]$ 
```

to resize the photo _/home/me/img/photo.jpg_

```
[me@linux ~]$ ls /home/me/img/
photo.jpg
[me@linux ~]$ podman run -ti --rm -v /home/me/img:/home/me/img:Z localhost/foobar:centos8 gm convert -resize 50% /home/me/img/photo.jpg /home/me/img/resized_photo.jpg
[me@linux ~]$ ls /home/me/img/
photo.jpg
photo_resized.jpg
[me@linux ~]$ 
``` 

Show the GraphicsMagick version

``` 
[me@linux ~]$ podman run --rm localhost/foobar:centos8 gm version | head -1
GraphicsMagick 1.3.34 2019-12-24 Q16 http://www.GraphicsMagick.org/
``` 




#### Example Ubuntu LTS : Resize a photo with GraphicsMagick

Let's use the [previously built](#example-ubuntu-lts--install-graphicsmagick-with-podman-build) container image __localhost/foobar:ubuntu1804__

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:ubuntu1804
REPOSITORY         TAG           IMAGE ID       CREATED          SIZE
localhost/foobar   ubuntu1804   891d0886ab51  1 hour ago   112 MB
[me@linux ~]$ 
```

to resize the photo _/home/me/img/photo.jpg_

```
[me@linux ~]$ ls /home/me/img/
photo.jpg
[me@linux ~]$ podman run -ti --rm -v /home/me/img:/home/me/img:Z localhost/foobar:ubuntu1804 gm convert -resize 50% /home/me/img/photo.jpg /home/me/img/resized_photo.jpg
[me@linux ~]$ ls /home/me/img/
photo.jpg
photo_resized.jpg
[me@linux ~]$ 
``` 

Show the GraphicsMagick version

``` 
[me@linux ~]$ podman run --rm localhost/foobar:ubuntu1804 gm version | head -1
GraphicsMagick 1.3.28 2018-01-20 Q16 http://www.GraphicsMagick.org/
``` 

#### Example Alpine : Resize a photo with GraphicsMagick

Let's use the [previously built](#example-alpine--install-graphicsmagick-with-podman-build) container image __localhost/foobar:alpine3__

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:alpine3
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
localhost/foobar   alpine3   93a5ac124e03   2 hours ago   24.8 MB
[me@linux ~]$ 
``` 

to resize the photo _/home/me/img/photo.jpg_

```
[me@linux ~]$ ls /home/me/img/
photo.jpg
[me@linux ~]$ podman run -ti --rm -v /home/me/img:/home/me/img:Z localhost/foobar:alpine3 gm convert -resize 50% /home/me/img/photo.jpg /home/me/img/resized_photo.jpg
[me@linux ~]$ ls /home/me/img/
photo.jpg
photo_resized.jpg
[me@linux ~]$ 
``` 

Show the GraphicsMagick version

``` 
[me@linux ~]$ podman run --rm localhost/foobar:alpine3 gm version | head -1
GraphicsMagick 1.3.33 2019-07-20 Q16 http://www.GraphicsMagick.org/
``` 

# Search for a pre-built container image

Instead of building your own container image you 
can often find a pre-built container image in one of the popular container image registries.

## Example: Search docker.io
Let's search for a pre-built container image that has the software GraphicsMagick installed.

A search for _GraphicsMagic_ on 
https://hub.docker.com/
results in a list of about 100 different container images from the registry _docker.io_.
The results are ordered in popularity.
One of the most popular at the time of writing is
https://hub.docker.com/r/jameskyburz/graphicsmagick-alpine/tags


It has a few different versions. To the right-hand side of the web page, you see for instance the text

`docker pull jameskyburz/graphicsmagick-alpine:v3.0.0`

That is the docker command to download ("pull") that version of the container image.

`podman` is aiming to be a drop-in replacement for the `docker` command.
When working with podman it is better to use the full path of the container image,
 i.e. always prepend "_docker.io/_" to the
container image names found at https://hub.docker.com/.

In other words, use

_docker.io/jameskyburz/graphicsmagick-alpine:v3.0.0_

instead of

_jameskyburz/graphicsmagick-alpine:v3.0.0_.

The corresponding podman command is therefore 

`podman pull docker.io/jameskyburz/graphicsmagick-alpine:v3.0.0`

To resize the photo _/home/me/img/photo.jpg_

```
[me@linux ~]$ ls /home/me/img/
photo.jpg
[me@linux ~]$ podman run -ti --rm -v /home/me/img:/home/me/img:Z docker.io/jameskyburz/graphicsmagick-alpine:v3.0.0 gm convert -resize 50% /home/me/img/photo.jpg /home/me/img/resized_photo.jpg
[me@linux ~]$ ls /home/me/img/
photo.jpg
photo_resized.jpg
[me@linux ~]$ 
``` 
# Security
## Consider the risc of malicious code in pre-built container images

Container images under _docker.io/library/*_ are official Docker library images.
Normally we should be able to trust those more than a container image published by
an arbitrary user at _docker.io/username/containername:tag_.

Luckily using `podman run` for executables of unknown origin in containers is safer

```
[me@linux ~]$ podman run -ti --rm example.com/hacker/malicious:v1 malicious_executable
```

than executing an untrusted executable directly

```
[me@linux ~]$ wget http://example.com/malicious_executable
[me@linux ~]$ chmod 755 malicious_executable
[me@linux ~]$ ./malicious_executable
```

The reason is that `podman run` provides extra encapsulation protection.

* Write access to directories on the host system need to be granted explicitly with (`--volume`, `-v`).
* add more items here ...

## How to run a command in a container image in a more secure and restricted way

A command for resizing photos should not need access to the network.
To disable network access, add `--net none` as an argument to `podman run`.

`podman run` can be restricted even further by for instance

* --read-only=true
* --security-opt=
* --ulimit=option
* --cap-drop=
* --cpu-quota=
* --shm-size=
* --http-proxy=false
* --volume (-v) can be made read-only
* --security-opt=seccomp=profile.json
* --security-opt=no-new-privileges

Man page for `podman run`:
https://github.com/containers/libpod/blob/master/docs/source/markdown/podman-run.1.md

## How to save disk space

### Easy tip: Use containers based on Alpine 

Alpine container images are very small. 

### Easy tip: Remove unnecessary container images

`podman rmi imagename`

Example: Removing container image _localhost/foobar:centos8_

```
[me@linux ~]$ podman images --filter reference=localhost/foobar:centos8
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
localhost/foobar   centos8   b1d1374ebbbf   31 minutes ago   395 MB
[me@linux ~]$ df -h --output=avail .
Avail
 9.0G
[me@linux ~]$ podman rmi localhost/foobar:centos8
Untagged: localhost/foobar:centos8
Deleted: b1d1374ebbbf650b34b81f0a3a3cb8cfba0a5b98da57582c70373d6abe695459
[me@linux ~]$ df -h --output=avail .
Avail
 9.2G
[me@linux ~]$ 
```

The command
`podman images --sort size`
lists the images in size order.

To list the biggest container image

```
[me@linux ~]$ podman images --sort size | tail -1
<none>                              <none>       a4977efc66f2   8 days ago     1.56 GB
[me@linux ~]$ 

```

## How to save time

### Speed up _podman build_ by reusing the package metadata cache

The first step of a container build is often to download metadata from
the package repositories and post-process the data.

This build step may take half a minute or so but luckily we can avoid it by reusing
the result.

The trick is to create the package metadata cache in advance and reuse it with an  _overlay mount_.

#### Example Fedora : Speed up _podman build_ by reusing the DNF metadata cache

Let's assume we are building containers based on Fedora 31.

First, create an empty directory, for instance _/home/me/f31cache_.

```
[me@linux ~]$ mkdir $HOME/f31cache
[me@linux ~]$ 
```

Fill the directory with the most recent __dnf__ metadata cache for _Fedora 31_.

```
[me@linux ~]$ time podman run --rm -v $HOME/f31cache:/var/cache/dnf:Z registry.fedoraproject.org/fedora:31 dnf makecache
Fedora Modular 31 - x86_64                      413 kB/s | 5.2 MB     00:12    
Fedora Modular 31 - x86_64 - Updates            3.0 MB/s | 4.0 MB     00:01    
Fedora 31 - x86_64 - Updates                     16 MB/s |  22 MB     00:01    
Fedora 31 - x86_64                               30 MB/s |  71 MB     00:02    
Last metadata expiration check: 0:00:01 ago on Sat Mar 21 18:50:20 2020.
Metadata cache created.

real	0m36.327s
user	0m0.152s
sys	0m0.076s
[me@linux ~]$ 
```

The command took __36__ seconds to finish.

```
[me@linux ~]$ du -sh $HOME/f31cache
212M	/home/me/f31cache
```

The directory consumes __212 MB__ of disk space.

Let's rebuild the [previously built](#example-fedora--install-graphicsmagick-with-podman-build) container image __localhost/foobar:fedora31__, but
now by reusing the DNF metadata cache from _/home/me/f31cache_.

Copy-paste these lines into the terminal

```
echo "FROM docker.io/library/fedora:31
RUN dnf -y update && dnf -y install epel-release && dnf -y install GraphicsMagick && dnf clean all
" | time podman build -v $HOME/f31cache:/var/cache/dnf:O -t foobar:fedora31 -

```

The `podman build` command finishes succesfully. `time` prints

```
0.17user 0.06system 0:00.22elapsed 104%CPU (0avgtext+0avgdata 42212maxresident)k
0inputs+3232outputs (0major+6383minor)pagefaults 0swaps
```

__0.22 seconds__! Less than one second! No, that is too fast to believe it's true.
We need to add the flag `--no-cache` so that podman will actually rebuild the container image
without reusing cached results.

A second trial:

Copy-paste these lines into the terminal

```
echo "FROM docker.io/library/fedora:31
RUN dnf -y update && dnf -y install epel-release && dnf -y install GraphicsMagick && dnf clean all
" | time podman build --no-cache -v $HOME/f31cache:/var/cache/dnf:O -t foobar:fedora31 -

```

The `podman build` command finishes succesfully. `time` prints

```
13.73user 3.20system 0:25.77elapsed 65%CPU (0avgtext+0avgdata 173404maxresident)k
553208inputs+523056outputs (453major+320854minor)pagefaults 0swaps
```

About __26 seconds__!

A normal build without `-v $HOME/f31y3:/var/cache/dnf:O` takes __52 seconds__.

__Conclusion:__ Reusing the DNF metadata cache speeds things up.

The blog post [_Speeding up container image builds with Buildah_](https://www.redhat.com/sysadmin/speeding-container-buildah) provides more details.

## Linux container images for the brave adventurous user not afraid of bugs

| Container image | Size (MB)   | Comment |
| ---- | --      | --      |
| registry.fedoraproject.org/fedora:rawhide | 207  | The latest development of Fedora (not yet released). :warning: Expect more bugs. |
| docker.io/library/alpine:edge | 6  | The latest development of Alpine (not yet released). :warning: Expect more bugs. |
| docker.io/library/debian:unstable | 123 | The latest development of Debian (not yet released). :warning: Expect more bugs. |

## When to use the flags _-i_ (_--interactive_) and _-t_ (_--tty_)

If your program reads from _stdin_, use the command line flag `-i`

```
[me@linux ~]$ echo Just one line | podman run --rm -i docker.io/library/ubuntu:18.04 wc -l
1
[me@linux ~]$ 
```

If your program needs interaction over the terminal, use the command line flags `-t` and `-i`

```
[me@linux ~]$ podman run --rm -ti docker.io/library/ubuntu:18.04 /bin/bash -c 'echo -n "Type something: "; read aa; echo "You typed: $aa"'
Type something: abc
You typed: abc
[me@linux ~]$ 
```

:warning: Using the `-t` flag may slightly modify the output, by adding extra _carriage return_ characters.
This might be very surprising. The extra _carrage return_ (`\r`) is not added by podman, but by
the terminal. 

```
[me@linux ~]$ echo abc | od -c
0000000   a   b   c  \n
0000004
[me@linux ~]$ podman run --rm docker.io/library/ubuntu:18.04 echo abc | od -c
0000000   a   b   c  \n
0000004
[me@linux ~]$ podman run --rm -ti docker.io/library/ubuntu:18.04 echo abc | od -c
0000000   a   b   c  \r  \n
0000005
[me@linux ~]$ 
```

## The professional way, using Dockerfile and GitHub/GitLab
### Using Github

Both [GitHub](https://github.com) and [GitLab](https://gitlab.com) provide a hosted version control system.
They are mainly used for hosting the source code of many software projects.

#### Create a public GitHub repo with a Dockerfile

First sign up for an (free) account on GitHub, if you haven't done so already. 
On the [front page](https://github.com/) you will see a green button labeled "_Sign up for GitHub_".

Then create a new repo
https://help.github.com/en/github/getting-started-with-github/create-a-repo

At step nr 4: _Choose a repository visbility._ choose _Public_.

At step nr 7: choose _Commit directly to the master branch_ instead of _Create a new branch for this commit and start a pull request_.

In the web interface, create a new file named _Dockerfile_.


Yet to be written ...
