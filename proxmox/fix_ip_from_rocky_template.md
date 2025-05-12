# Note here to fix the IP address of a Rocky VM from a template

Important : When you create your template, **NOTE YOUR ROOT PASSWORD**.

For example : `p0w3rLa8z!`.

There are many web tutorials to use and create good templates on PVE.

Now, when you need to fix the VM IP, you need these commands :

```bash
# ens18 is the default interface on a new Rocky
sudo nmcli connection modify ens18 ipv4.addresses 192.168.1.201/24
sudo nmcli connection modify ens18 ipv4.method manual
sudo nmcli connection modify ens18 ipv4.gateway 192.168.1.254
sudo nmcli connection modify ens18 ipv4.dns "192.168.1.254 8.8.8.8"
sudo nmcli connection down ens18 && sudo nmcli connection up ens18
```
