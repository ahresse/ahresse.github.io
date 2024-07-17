---
layout: post
title:  "Setting-up a local Debian repository"
date:   2024-07-17 14:00:00 +0200
categories: debian packaging
---

In order to develop Debian packages and test these locally or on my DUT (Device Under Test), I usually trend to setup a local APT repository.

## 1 - Edit APT sources on DUT:

On the environment you would like to test, create a new file:

```
sudo vim /etc/apt/source.list.d/local.sources
```

With the content as follow:

```
Types: deb
URIs: http://<server-ip>:8000
Suites: <codename>
Components: main
Trusted: true
```

Replacing:
 - `<server-ip>` with the IP adress of you development machine.
 - `<codename>` with the suite codename you are testing (eg. `oracular`).

*The `Trusted` field is here to simplify the setup but you can also specify the path of your public keys (`.asc` files) your previously imported on the DUT for more security with the `Signed-By` field (eg. `Signed-By: /usr/share/keyrings/<my-key>.asc`).*

## 2 - Generate your binary packages:

I am using `git-buildpackage` to create my debian packages.

In addition, I set a target directory for these to land in a dedicated folder. To do so, I have these fields in my `~/.gbp.conf`:

```
[buildpackage]
export-dir = ../build-area
tarball-dir = ../tarballs
```

Then, after building my binary packages, I endup with a buch of `.deb` files in the `../build-area` directory.

## 3 - Create Packages index files

Within this `../build-area` directory, I will setup the root of my APT server:
```
cd ../build-area
mkdir -p dists/<codename>/main/binary-<arch>/
```

Then, generate the packages index and the compressed version of it:
```
dpkg-scanpackages --arch <arch> . > dists/<codename>/main/binary-<arch>/Packages
cat dists/<codename>/main/binary-<arch>/Packages | gzip -9 > dists/<codename>/main/binary-<arch>/Packages.gz
```

Replacing:
 - `<codename>` with the suite codename you are testing (eg. `oracular`).
 - `<arch>` with the architecture you're targeting (eg. `amd64`).
 
## 4 - Launch an HTTP server acting for your APT server

Still with the `build-are` directory of your development system, launch and HTTP server:

```
python3 -m http.server
```

## 5 - Get packages on your DUT

On your DUT, get packages as usual:

```
sudo apt update
sudo apt install <your-package>
```

## Going further

In order to make the development process even smoother, I added the package index update within the `postbuild` steps of `git-buildpackage`. Here is what my `~/.gbp.conf` `postbuild` field looks like for `amd64` packages under Ubuntu Oracular:

```
postbuild = lintian -I $GBP_CHANGES_FILE && \
  echo "Lintian OK" && \
  mkdir -p ../build-area/dists/oracular/main/binary-amd64/ && \
  dpkg-scanpackages --arch amd64 ../build-area > ../build-area/dists/oracular/main/binary-amd64/Packages && \
  cat ../build-area/dists/oracular/main/binary-amd64/Packages | gzip -9 > ../build-area/dists/oracular/main/binary-amd64/Packages.gz
```

