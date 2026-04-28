# "Interconnect" Demo for Latitude.sh

<p align="center">
    <img src="https://raw.githubusercontent.com/dlotterman/megaport_code_snippets/refs/heads/main/demo/interconnect/assets/lsh_interconnect_demo.png" alt="interconnect diagram" width="750">
  </a>
</p>

## Magic Numbers

The "Interconnect" demo environment is based out of Dallas, where a "Latitude.sh Labs" org [VLAN](https://www.latitude.sh/docs/networking/private-networks) has been plumbed into a Megaport [MCR](https://docs.megaport.com/mcr/). The magic numbers around that interconnection are:

LSH VLAN:`2105`
LSH Subnet: `172.31.0.0/18`
LSH Gateway: `172.31.0.1`
LSH Peering IP: `172.31.254.241/31`
LSH BGP ASN: `65213`
MCR Peering IP: `172.31.254.242/31`
MCR BGP ASN: `4200000001`

## The Demo

Start by showing the customer the LSH VLAN `latitude-sa-mcr-da-1-172-31-0-1/18`. Then show the diagram in this repo, and talk about how we use Megaport VXCs to "extend" an LSH VLAN, in this case to an MCR. This is a Layer-3 oriented connection where the LSH Network has been configured to BGP Peer into the MCR. The LSH Network provides access in the form of a Gateway IP in that VLAN.

Then go to the Megaport portal and search services for `latitude-sa-mcr-da-1`, this will bring up the MCR and its VXC into LSH. Bring up VXC details to show the MCR Interface IP and BGP status.

Go back to the LSH interface.

Launch an LSH instance with Ubuntu 24.04 in the "Latitude.sh Labs" org in Dallas, it must be an instance with two ports (not 1x). Optionally, add a unique numerical, non-padded string as a suffix to the hostname to take advantage of a little copy+paste simplicity later on, for example "mcr-demo-7" or "mcr-demo-187".

When the instance is available or "ON", apply the `latitude-sa-mcr-da-1-172-31-0-1/18` VLAN to the instance in it's "Network" page.

SSH into the instance and copy and paste the below command to "capture" the numerical suffix as a shell variable:

```
HOSTNAME_SUFFIX=$(hostname | awk -F "-" '{print$NF}')
```

Then write the [netplan](https://netplan.readthedocs.io/en/stable/) file for the VLAN interface. Note that the below template already includes the relevant Magic Numbers (see above), and if you are not following the "suffix" cheat, you will need to edit in your own IP information where the `$HOSTNAME_SUFFIX` variable is currently.

```
cat > /etc/netplan/55-eno2-vlan-2105.yaml << EOL
network:
  version: 2
  renderer: networkd
  vlans:
    vlan.2105:
      id: 2105
      link: eno2
      addresses: [172.31.0.$HOSTNAME_SUFFIX/18]
      routes:
        - to: 172.16.0.0/12
          via: 172.31.0.1
EOL
```

If you want, look at the `cloud-init` written file that names `eno1` and `eno2`, note there is no need to edit this file.

```
cat /etc/netplan/50-cloud-init.yaml
```

Fix permissions and apply (ignore output about `gateway4/6`, that is a known transition:

```
chmod 0700 /etc/netplan/*
netplan apply
```

Now ping the IP of the LSH Gateway inside VLAN `2105`:

```
ping 172.31.0.1
```


Should yield output like:
```
# ping 172.31.0.1
PING 172.31.0.1 (172.31.0.1) 56(84) bytes of data.
64 bytes from 172.31.0.1: icmp_seq=1 ttl=255 time=0.588 ms
```

Note if the ping is returning unreachable, your a wizard and moving fast, check the LSH instance's "Networking" details page, the VLAN is likely still in the "Connecting" state. It can take up to ~3min at the worst, usually a handful of seconds.

Then ping the LSH Peering IP, this is essentially the IP on the Latitude.sh router that brings in Megaport Connectivity:

```
ping 172.31.254.241
```

Then ping the MCR:
```
172.31.254.242
```

## Celebrate

Congratulations, you've proved you can connect to anything you can connect to via Megaport, your a Wizard Harry.
