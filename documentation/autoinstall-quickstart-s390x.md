# Autoinstall Quick Start for s390x

The intent of this page is to provide simple instructions to perform an autoinstall in a VM on your machine on s390x.

This page is just a slightly adapted page of [the autoinstall quickstart page](autoinstall-quickstart.md) mapped to s390x.

## Download an ISO

At the time of writing (just after the kinetic release), the best place to go is here:
<https://cdimage.ubuntu.com/ubuntu/releases/22.10/release/>

<pre><code>wget https://cdimage.ubuntu.com/ubuntu/releases/22.10/release/ubuntu-22.10-live-server-s390x.iso -P ~/Downloads</code></pre>

## Mount the ISO

<pre><code>mkdir -p ~/iso
sudo mount -r ~/Downloads/ubuntu-22.10-live-server-s390x.iso ~/iso</code></pre>

## Write your autoinstall config

This means creating cloud-init config as follows:

<pre><code>mkdir -p ~/www
cd ~/www
cat > user-data << 'EOF'
#cloud-config
autoinstall:
  version: 1
  identity:
    hostname: ubuntu-server
    password: "$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0"
    username: ubuntu
EOF
touch meta-data</code></pre>

The crypted password is just "ubuntu".

## Serve the cloud-init config over http

Leave this running in one terminal window:

<pre><code>cd ~/www
python3 -m http.server 3003</code></pre>

## Create a target disk

Proceed with a second terminal window:

<pre><code>sudo apt install qemu-utils
...</code></pre>

<pre><code>qemu-img create -f qcow2 disk-image.qcow2 10G
Formatting 'disk-image.qcow2', fmt=qcow2 size=10737418240 cluster_size=65536 lazy_refcounts=off refcount_bits=16

qemu-img info disk-image.qcow2
image: disk-image.qcow2
file format: qcow2
virtual size: 10 GiB (10737418240 bytes)
disk size: 196 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false</code></pre>

## Run the install!

<pre><code>sudo apt install qemu-kvm
...</code></pre>

You may need to add the default user to the kvm group:  <<BR>>
`sudo usermod -a -G kvm ubuntu   # re-login to make the changes take effect`

<pre><code>kvm -no-reboot -name auto-inst-test -nographic -m 2048 \
    -drive file=disk-image.qcow2,format=qcow2,cache=none,if=virtio \
    -cdrom ~/Downloads/ubuntu-22.10-live-server-s390x.iso \
    -kernel ~/iso/boot/kernel.ubuntu \
    -initrd ~/iso/boot/initrd.ubuntu \
    -append 'autoinstall ds=nocloud-net;s=http://_gateway:3003/ console=ttysclp0'</code></pre>

This will boot, download the config from the server set up in the previous step and run the install.
The installer reboots at the end but the -no-reboot flag to kvm means that kvm will exit when this happens.
It should take about 5 minutes.

## Boot the installed system

<pre><code>kvm -no-reboot -name auto-inst-test -nographic -m 2048 \
    -drive file=disk-image.qcow2,format=qcow2,cache=none,if=virtio</code></pre>

This will boot into the freshly installed system and you should be able to log in as ubuntu/ubuntu.
