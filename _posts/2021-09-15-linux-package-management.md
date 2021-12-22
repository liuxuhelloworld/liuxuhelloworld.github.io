Package management is a method of installing and maintaining software on the system.

Linux阵营主要有两大packaging system：
- the Debian .deb: Debian, Ubuntu, Linux Mint, Raspbian
- the RedHat .rpm: Fedora, CentOS, RedHat, OpenSUSE

A package file is a compressed collection of files that comprise the software package.
- programs and data files
- metadata
- pre- and post-installation scripts

一般的，distribution vendors会维护a central repository，向这个distribution的用户提供可用的packages.

Package tools:
- low-level tools: handle tasks such as installing and removing package files, dpkg(Debian-style), rpm(RedHat-style)
- high-level tools: metadata searching and dependency resolution, apt-get(Debian-style), yum(RedHat-style)