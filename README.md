# mkosi-built Images for systemd-machined

Minimal OS container images built with [**mkosi**](https://github.com/systemd/mkosi) for **systemd-machined**.

## Available images

| Distro | Images |
|--------|--------|
| CentOS | `centos-10` |
| Fedora | `fedora-44`, `fedora-rawhide` |
| RHEL UBI | `rhel-ubi-10` |

All images are available for both x86-64 and arm64 architectures.

## Usage

### Import an image

#### Using systemd-sysupdate

Define a helper function:

```sh
import_image() {
  IMAGE=$1
  TAG=${IMAGE}_$(systemctl show -P Architecture)
  URL=https://github.com/albertescanes/nspawn-images/releases/download
  run0 install -d -m 0755 -o 0 -g 0 /etc/sysupdate.${IMAGE}.d
  curl -sfL $URL/$TAG/sysupdate.transfer \
    | run0 tee /etc/sysupdate.${IMAGE}.d/sysupdate.transfer
  updatectl update component:${IMAGE}
}
```

Then import one of the images listed in the table above:

```sh
import_image <image>
```

The function will:
- Download the sysupdate transfer definition for your architecture.
- Install it to `/etc/sysupdate.<image>.d/`.
- Run `updatectl update` to download and import the container image.

#### Importing manually

If you prefer to import the image manually without sysupdate, you can download the tarball directly from the [releases](../../releases) and use importctl:

```sh
run0 importctl import-tar -m <tarball>
```

### Start and enter the container

```sh
run0 machinectl start <image>
run0 machinectl shell <image>
```

### Use host networking

To disable private networking and make the container use host networking instead, create a `.nspawn` settings file for the container:

```sh
run0 machinectl edit <image>
```

This opens an editor for `/etc/systemd/nspawn/<image>.nspawn`. Add the following:

```ini
[Network]
VirtualEthernet=no
```

Then restart the container machine for the change to take effect:

```sh
run0 machinectl reboot <image>
```

## Updating an image

If you imported the image using systemd-sysupdate, you can update it to the latest version by simply running:

```sh
updatectl update component:<image>
```
