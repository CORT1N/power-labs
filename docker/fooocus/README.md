# Troubleshooting
## Docker in LXC container

In my case, I decided to rely on my existing infrastructure, so a Proxmox cluster, on which I mount LXC Containers or VMs.

One of these containers, which I gave more power to, is used entirely for what I do in Docker.

In order to do generative AI in a Docker container running in this environment, it needs to be able to use the hardware GPU on the Proxmox host that its own LXC container is on.

This part was the complicated one.

### First, the drivers had to be installed on the Proxmox host.
- Download the appropriate GPU driver from the Nvidia website (*.run*)
- If when it's launched it gives us some errors, fix them
- Following this, the `nvidia-smi` command is supposed to return us information about the GPU it recognizes

### Second, the LXC container configuration needed to be modified to accommodate the GPU.

To do this, you need to modify the file associated with the **id** of the container with `nano /etc/pve/lxc/<id>.conf` and add:

```conf
lxc.cgroup2.devices.allow: a
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

To mount the Nvidia kernel files in my case, and let the container access the *cgroup2* of the host machine.

### Third, you need to install the same driver on the LXC container

With a small particularity, very complicated to find on Reddit: `./RUN[whatever_driver_name] --no-kernel-module` **because they are already loaded in the mount that we made earlier.**, but you still have to install the driver on the container.

Finally for this part, you have to restart the LXC container and test the `nvidia-smi` command and expect for the same output.

### Fourth, you need to configure Docker to use the Nvidia runtime

First, you must have installed the **nvidia container toolkit** on the LXC container.

Then set the *no-cgroups* tag to true and uncomment it in `nano /etc/nvidia-container-runtime/config.toml` on the LXC container.

Finally:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
cat /etc/docker/daemon.json
```

If the output is not empty, and contains an *nvidia runtime*, all you have to do is test: `docker run --gpus all nvidia/cuda:12.6.1-base-ubuntu24.04 nvidia-smi`.

This command will run the `nvidia-smi` command in a Docker container and should return the same output as on the Proxmox host and LXC container.