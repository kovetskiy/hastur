![hastur](https://cloud.githubusercontent.com/assets/674812/10144792/fadcbd28-660e-11e5-8569-894e874cff14.png)

hastur is a tool for launching systemd-nspawn containers without the need for
manual configuration.

It will setup networking, a base root FS and even an overlay FS for containers
automatically.

The primary usecase for hastur is supporting testcases for distributed systems
and running a local set of trusted containers.

![gif](https://cloud.githubusercontent.com/assets/674812/10140037/37f12ea2-65f5-11e5-90c7-eb18e6a9b37b.gif)

# Motivation

systemd-nspawn is useful tool, which can create and run lightweight containers
without any additional software, because it's available out-of-the-box systemd.

However, it requires some configuration to run a working container, such as
managing the network configuration, and downloading and extracting packages.

hastur offers all this configuration automatically, which makes it possible to
run fully-configured systemd-nspawn containers in seconds.

# Installation

## Arch Linux

hastur is available for Arch Linux (for now, only) through the AUR:

https://aur4.archlinux.org/packages/hastur/

## go get

hastur is also go-gettable:

```
go get github.com/seletskiy/hastur
```

# Usage

## Testing water

The most simple usage is testing hastur out-of-the-box: you can tell it to
create an ephemeral container with a basic set of packages:

```
sudo hastur -S
```

After invoking that command, you will end up ~~in Cthulhu's void~~ in the bash
shell.

This test container will be deleted after you exit its shell.

## Creating non-ephemeral containers

Passing the `-k` flag will tell hastur to keep the container after exit:

```
sudo hastur -kS
```

If you don't like the fantastical autogenerated names, you can pass the flag
`-n`:

```
sudo hastur -Sn my-cool-name
```

Note that ephemeral containers are only the ones that both have autogenerated
names and were not started with the `-k` flag.

## Networking

hastur will take care of setting up the networking by creating a bridge and
setting up a shared network. By default, hastur will automatically generate IP
addresses, and you can see a container's address either in its starting message
or by running the query command:

```
sudo hastur -Q
```

You can specify your own IP address by passing the `-a` flag, like this:

```
sudo hastur -S -a 10.0.0.2/8
```

## But what about software?

hastur uses package-based container configurations and will happily populate
your container with the packages that you want. You can use the `-p` flag for
this:

```
sudo hastur -S -p nginx
```

That will create a container with the `nginx` package pre-installed. In
actuality, hastur uses overlays to keep base dirs separate from container data.
The base dirs, or, if you like, images, are just prepared root filesystems,
which have pre-installed packages. You can query the cached base dirs by
running hastur with the `-Qi` flag:

```
sudo hastur -Qi
```

In fact, from hastur's standpoint, a container is just a data dir, which gets
overlayed on top of a root filesystem and then given network capability, so
it will not remember what IP address a container has or what set of packages it
has installed if you forget to specify the correct options.

For example:

```
sudo hastur -Sn test -p git -- /bin/git --version
```

Will output:

```
git version 2.5.3
```

But running this container the next time without `-p git` will tell you that
git is not installed:

```
sudo hastur -Sn test -- /bin/git --version
```

This outputs:

```
No such file or directory
```

However, this `test` container will have a separate FS and all data files will
persist across the two runs.

# Additional information

hastur can operate over several root directories and keep container instances
separately from each other. The `-r` flag is used for this:

```
sudo hastur -r solar-system -Sn earth
sudo hastur -r solar-system -Sn moon
sudo hastur -r alpha-centauri -Sn a
sudo hastur -r alpha-centauri -Sn b
sudo hastur -r alpha-centauri -Sn c
```

The output will be different for these two commands:

```
sudo hastur -r solar-system -Q
sudo hastur -r alpha-centauri -Q
```

The first one will list only the `earth` and `moon` containers, and the second
will only list the `a`, `b` and `c` containers.

# License

MIT.
