# Digital Ocean: Alpine Linux

This guide instructs you on how to put [Alpine Linux](http://alpinelinux.org/) on a [Digital Ocean](https://www.digitalocean.com/?refcode=a0f96edad652) (referral link) droplet.

## Requirements

- Digital Ocean account
- Local Docker installation (used to generate the Alpine root filesystem)

## Process

### Generate Alpine root file system

1. Ensure Docker is running.
2. Download and unzip [`gliderlabs/docker-alpine`](https://github.com/gliderlabs/docker-alpine).
    - `wget -O docker-alpine-master.zip https://github.com/gliderlabs/docker-alpine/archive/master.zip`
    - `unzip docker-alpine-master.zip`
3. Build the builder.
    - `docker build -t docker-alpine-builder docker-alpine-master/builder/`
4. Build the root file system (change `v3.3` to the Alpine version you want to build).
    - `docker run --name alpine-builder docker-alpine-builder -r v3.3`
5. Copy the root file system from the container.
    - `docker cp alpine-builder:/rootfs.tar.gz .`
6. (Optional) Clean up builder.
    - `docker rm alpine-builder`
    - `docker rmi docker-alpine-builder`
    - `rm -rf docker-alpine-master{,.zip}`

You should now have `rootfs.tar.gz` in your current directory.

### Prepare droplet

1. Create droplet.
    - In your Digital Ocean control panel, click "Create Droplet".
    - Fill in your information, select the "Debian 7.9 x64" image, and click "Create Droplet".
2. Transfer `rootfs.tar.gz` to the droplet.
    - `scp rootfs.tar.gz root@<IP address>:`
3. SSH into the droplet.
    - `ssh root@<IP address>`
4. Inside the droplet, extract the Alpine files onto your hard drive.
    - `mkdir /alpine`
    - `tar xf rootfs.tar.gz -C /alpine`
    - `poweroff`

### Setup root file system

1. In your Digital Ocean droplet control panel, click "Settings".
2. Under the "Recovery" tab, click "Mount Recovery Kernel".
3. Under "Power", click "Power On".
4. Under "Access", click "Console Access".
5. In the recovery console, move the extracted Alpine files into the root of your drive.
    - `mkdir /mnt`
    - `mount -t ext4 /dev/vda1 /mnt`
    - `mv /mnt/etc/network/interfaces /mnt/alpine/etc/network/`
    - `mv /mnt/root/.ssh/ /mnt/alpine/root/`
    - `mv /mnt/etc/fstab /mnt/alpine/etc/`
    - `mv /mnt/alpine/ /tmp/`
    - `rm -rf /mnt/*`
    - `mv /tmp/alpine/* /mnt/`
    - `umount /mnt/`
    - `poweroff`
6. Click "Back to Droplet".
7. Under "Settings", click "Kernel" tab.
8. Select "Original Kernel", then click "Change".
9. Under "Power", click "Power On".

### Configuring Alpine

1. Re-open "Console Access".
2. Login as `root`.
3. Enable writing to the file system.
    - `mount -o rw,remount /dev/vda1 /`
    - `vi /etc/fstab`
    - Set the 4th column to `defaults`
    - Save and exit.
4. Configure networking and SSH.
    - `setup-hostname`
    - `setup-dns`
    - `service networking restart`
    - `apk update`
    - `setup-sshd`
5. Enable services.
    - `rc-update add hostname boot`
    - `rc-update add networking boot`
    - `rc-update add urandom boot`
    - `rc-update add cron`
    - `rc-update add swap boot`
6. Reboot.
    - `reboot`

After removing the old SSH fingerprint from your local machine's `~/.ssh/known_hosts` file, you should now be able to SSH into your droplet.

## License

[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)

## Author

Tim Cooper (<tim.cooper@layeh.com>)
