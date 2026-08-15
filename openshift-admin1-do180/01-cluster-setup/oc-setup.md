Absolutely. Below is the **complete chronological documentation of what we did**, including the original disk problem, CRC installation/setup, disk expansion, system networking, HAProxy, firewall, and Windows hosts-file configuration.

I’ve kept it in the **same order you performed the work**, so you can save this as your setup/runbook.

# OpenShift CRC 4.22.1 on RHEL — Complete Setup Documentation

## 1. Environment

### RHEL server

```text
OS:                 Red Hat Enterprise Linux
Server LAN IP:      192.168.1.2
Network interface:  enp1s0
Gateway:            192.168.1.1
```

### CRC

```text
CRC version:        2.62.0
OpenShift bundle:   4.22.1
Architecture:       amd64
Hypervisor:         libvirt/KVM
Network mode:       system
CRC VM IP:          192.168.130.11
```

CRC's system-mode networking on Linux uses an internal network separate from the host's normal LAN network. Therefore:

```text
RHEL server = 192.168.1.2
CRC VM      = 192.168.130.11
```

The RHEL server's `192.168.1.2` address is **not changed**. CRC's `192.168.130.x` address is an internal CRC network address. ([CRC.dev][1])

---

# 2. Initial CRC setup

CRC was extracted under:

```text
/home/admin/crc-linux-2.62.0-amd64
```

The CRC bundle was cached at:

```text
/home/admin/.crc/cache/crc_libvirt_4.22.1_amd64.crcbundle
```

The bundle size was approximately:

```text
6.51 GiB
```

CRC setup automatically installed/configured:

* libvirt
* `crc-driver-libvirt`
* KVM/vsock support
* CRC admin helper
* CRC systemd services
* required udev configuration

The setup eventually reported:

```text
Your system is correctly setup for using CRC. Use 'crc start' to start the instance
```

CRC on RHEL requires libvirt and NetworkManager, and OpenShift CRC requires at least 4 physical CPU cores, 10.5 GB free memory and 35 GB storage. ([CRC.dev][2])

---

# 3. First CRC start

We ran:

```bash
crc start
```

CRC requested the Red Hat pull secret.

The pull secret was entered from the Red Hat console.

CRC then created the VM and initially reported:

```text
CRC instance is running with IP 127.0.0.1
```

At that point CRC was using the default **user network mode**.

---

# 4. CRC stopped because the RHEL root filesystem was full

CRC subsequently reported:

```text
ERRO Cannot get machine state: unexpected libvirt status 3
```

We investigated libvirt:

```bash
sudo virsh dominfo crc
```

The important error was:

```text
State: paused

Messages:
I/O error: disk='vda'
message='No space left on device'
```

So the problem was **not libvirt itself**.

The RHEL root filesystem was full.

---

# 5. Disk investigation

We checked:

```bash
df -h
df -ih
```

Initially:

```text
/dev/mapper/rhel-root   35G   35G   1.4M   100% /
```

The important discovery was that the **virtual disk itself was 80 GB**, but only ~39 GB had been partitioned for LVM.

We confirmed:

```bash
lsblk
```

Initially:

```text
vda             80G
├─vda1           1G       /boot
└─vda2          39G       LVM
  ├─rhel-root  35.1G      /
  └─rhel-swap   3.9G
```

`fdisk` confirmed:

```text
Disk /dev/vda: 80 GiB

/dev/vda1   1G
/dev/vda2  39G
```

So:

```text
Physical disk = 80 GB
Partitioned   = ~40 GB
Root FS       = ~35 GB
Unused disk   = ~40 GB
```

---

# 6. CRC was consuming significant disk space

We checked:

```bash
du -sh /home/admin/.crc
du -sh /home/admin/.crc/machines/crc
```

Result:

```text
33G /home/admin/.crc
398M /home/admin/.crc/machines/crc
```

Further investigation showed:

```text
/home/admin/.crc
├── cache       ~32G
├── machines    ~398M
├── bin
├── logs
└── other CRC files
```

The large cache was expected because the CRC bundle itself and extracted artifacts consume substantial space.

---

# 7. Install `growpart`

`growpart` was not initially installed:

```bash
command -v growpart
```

No output.

We found the package using:

```bash
sudo dnf provides '*/growpart'
```

It returned:

```text
cloud-utils-growpart
```

We installed it:

```bash
sudo dnf install -y cloud-utils-growpart
```

Then:

```bash
command -v growpart
```

returned:

```text
/usr/bin/growpart
```

The package's `--version` option wasn't supported, so:

```bash
growpart --version
```

returned:

```text
FAILED: unknown option --version
```

That was harmless.

---

# 8. Expand `/dev/vda2`

We confirmed that `/dev/vda2` stopped at approximately 39 GB even though `/dev/vda` was 80 GB.

We expanded partition 2:

```bash
sudo growpart /dev/vda 2
```

Output:

```text
CHANGED: partition=2
start=2099200
old: size=81786880 end=83886079
new: size=165672927 end=167772126
```

This expanded `/dev/vda2` to consume almost the entire 80 GB disk.

---

# 9. Resize the LVM physical volume

Next:

```bash
sudo pvresize /dev/vda2
```

Result:

```text
Physical volume "/dev/vda2" changed
1 physical volume(s) resized or updated / 0 physical volumes not resized
```

We verified:

```bash
sudo vgs
sudo pvs
```

Result:

```text
VG   VSize   VFree
rhel <79.00g 40.00g
```

So approximately 40 GB became available to LVM.

---

# 10. Expand the root logical volume

We assigned all available LVM space to root:

```bash
sudo lvextend -l +100%FREE /dev/mapper/rhel-root
```

Result:

```text
Size of logical volume rhel/root changed
from 35.05 GiB
to 75.05 GiB
```

The swap LV remained approximately:

```text
3.95 GiB
```

---

# 11. Expand the XFS filesystem

Because `/` uses XFS:

```text
/dev/mapper/rhel-root xfs /
```

we ran:

```bash
sudo xfs_growfs /
```

The filesystem expanded successfully.

We verified:

```bash
df -h /
```

Result:

```text
/dev/mapper/rhel-root
Size: 75G
Used: 36G
Avail: 40G
Use%: 48%
```

So the original disk problem was solved.

Final disk layout:

```text
/dev/vda       80G
├── /dev/vda1   1G   /boot
└── /dev/vda2  79G   LVM
    ├── root   75G   /
    └── swap    4G
```

---

# 12. CRC was successfully running again

We checked:

```bash
sudo virsh domstate crc
```

Result:

```text
running
```

Then:

```bash
crc status
```

Initially:

```text
CRC VM:          Running
OpenShift:       Starting (v4.22.1)
RAM Usage:       ...
Disk Usage:      ...
Cache Usage:     34.13GB
Cache Directory: /home/admin/.crc/cache
```

The CRC cache being ~34 GB was **not itself a problem anymore**, because the RHEL root filesystem had been expanded to ~75 GB.

---

# 13. CRC console URL

We checked:

```bash
crc console --url
```

It returned:

```text
https://console-openshift-console.apps-crc.testing
```

However, at this stage CRC was still using user-mode networking, meaning it was effectively local to the RHEL machine.

---

# 14. Verify server networking

We checked:

```bash
ip -br addr
```

Important output:

```text
enp1s0   UP   192.168.1.2/24
virbr0   DOWN 192.168.122.1/24
```

Routing:

```text
default via 192.168.1.1 dev enp1s0
```

Therefore the RHEL server's LAN address was:

```text
192.168.1.2
```

We also verified:

```bash
sudo firewall-cmd --state
```

Result:

```text
running
```

---

# 15. Check CRC network mode

Initially:

```bash
crc config get network-mode
```

returned:

```text
Configuration property 'network-mode' is not set.
Default value 'user' is used
```

We needed Windows to access the CRC instance through the RHEL server's LAN IP.

Therefore we changed CRC to **system network mode**.

---

# 16. Stop CRC

We ran:

```bash
crc stop
```

CRC stopped successfully.

---

# 17. Change CRC to system networking

We ran:

```bash
crc config set network-mode system
```

CRC reported:

```text
Network mode changed.
Please run `crc cleanup` and `crc setup`.
```

We then attempted `crc start` immediately and received:

```text
ERRO file not found:
/etc/NetworkManager/conf.d/crc-nm-dnsmasq.conf
```

This happened because changing the network mode requires the CRC setup to be regenerated.

---

# 18. CRC cleanup/setup for network-mode change

We followed the CRC-required process:

```bash
crc cleanup
```

then:

```bash
crc setup
```

After setup, we checked:

```bash
crc config get network-mode
```

Result:

```text
network-mode : system
```

At that moment `crc ip` returned:

```text
ERRO Machine does not exist.
Use 'crc start' to create it
```

That was expected because the CRC VM had been removed/recreated as part of the cleanup/setup process.

---

# 19. Start CRC using system networking

We ran:

```bash
crc start
```

After startup:

```bash
crc status
```

reported:

```text
CRC VM:          Running
OpenShift:       Running (v4.22.1)
```

Then:

```bash
crc ip
```

returned:

```text
192.168.130.11
```

This is the **CRC internal IP**.

It did NOT replace the RHEL server IP.

The final network arrangement is:

```text
RHEL server:
192.168.1.2

CRC VM:
192.168.130.11
```

CRC's documentation confirms that system-mode Linux networking uses the internal `192.168.130.x` network. ([CRC.dev][1])

---

# 20. Verify CRC API directly

We tested:

```bash
curl -k https://192.168.130.11:6443/version
```

It returned Kubernetes/OpenShift API version information:

```text
major: 1
minor: 35
gitVersion: v1.35.5
```

Therefore:

```text
RHEL → CRC API
```

was working correctly.

---

# 21. Verify CRC HTTPS endpoint

We tested:

```bash
curl -k -I https://192.168.130.11
```

It returned:

```text
HTTP/1.0 503 Service Unavailable
```

This is not evidence that CRC is broken.

The OpenShift ingress requires the appropriate hostname to route requests. Directly requesting the IP does not provide the expected OpenShift hostname.

---

# 22. Install HAProxy

We checked:

```bash
rpm -q haproxy
```

Initially:

```text
package haproxy is not installed
```

We installed it:

```bash
sudo dnf install -y haproxy
```

CRC's official remote-server procedure uses HAProxy to expose ports 80, 443 and 6443 from the RHEL server to the CRC VM. ([CRC.dev][1])

---

# 23. Back up HAProxy configuration

Before modifying the existing RHEL HAProxy configuration:

```bash
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

The original configuration was the standard RHEL example configuration using port 5000 and local test backends.

We replaced it with the CRC configuration.

---

# 24. Configure HAProxy for CRC

We used the CRC IP:

```text
192.168.130.11
```

The resulting configuration was:

```haproxy
global
    log /dev/log local0

defaults
    balance roundrobin
    log global
    maxconn 100
    mode tcp
    timeout connect 5s
    timeout client 500s
    timeout server 500s

listen apps
    bind 0.0.0.0:80
    server crcvm 192.168.130.11:80 check

listen apps_ssl
    bind 0.0.0.0:443
    server crcvm 192.168.130.11:443 check

listen api
    bind 0.0.0.0:6443
    server crcvm 192.168.130.11:6443 check
```

The official CRC remote-server documentation uses this same architecture: HAProxy listens on ports 80, 443 and 6443 and forwards them to the CRC VM. ([CRC.dev][1])

---

# 25. SELinux configuration for port 6443

Because RHEL uses SELinux, HAProxy needs permission to listen on TCP 6443.

The required utilities were installed:

```bash
sudo dnf install -y policycoreutils-python-utils
```

Then:

```bash
sudo semanage port -a -t http_port_t -p tcp 6443
```

This is part of the official CRC remote-server procedure. ([CRC.dev][1])

---

# 26. Validate HAProxy

Before starting HAProxy:

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

The configuration must pass validation.

Expected:

```text
Configuration file is valid
```

---

# 27. Configure RHEL firewall

We allowed the required OpenShift services:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=kube-apiserver
sudo firewall-cmd --reload
```

The official CRC remote-server documentation specifies these firewall services. ([CRC.dev][1])

The resulting purpose is:

```text
TCP 80    → OpenShift application HTTP
TCP 443   → OpenShift HTTPS/console
TCP 6443  → OpenShift/Kubernetes API
```

We did **not** need to change the RHEL server's IP address.

---

# 28. Enable/start HAProxy

HAProxy was started:

```bash
sudo systemctl enable --now haproxy
```

Verify:

```bash
sudo systemctl status haproxy --no-pager
```

And:

```bash
sudo ss -lntp | grep -E ':(80|443|6443)\b'
```

The expected architecture is:

```text
192.168.1.2:80
       ↓
192.168.130.11:80

192.168.1.2:443
       ↓
192.168.130.11:443

192.168.1.2:6443
       ↓
192.168.130.11:6443
```

---

# 29. Windows hosts file

The Windows computer is on the same LAN and reaches the RHEL server at:

```text
192.168.1.2
```

Therefore the Windows hosts file is:

```text
C:\Windows\System32\drivers\etc\hosts
```

We added:

```text
192.168.1.2 console-openshift-console.apps-crc.testing
192.168.1.2 api.crc.testing
192.168.1.2 oauth-openshift.apps-crc.testing
```

### Why `oauth-openshift` was necessary

When opening the console, OpenShift redirected the browser to:

```text
oauth-openshift.apps-crc.testing
```

Windows didn't know that hostname and showed:

```text
DNS_PROBE_FINISHED_NXDOMAIN
```

Therefore we added:

```text
192.168.1.2 oauth-openshift.apps-crc.testing
```

CRC uses two DNS domains:

```text
crc.testing
apps-crc.testing
```

with examples including:

```text
api.crc.testing
console-openshift-console.apps-crc.testing
```

CRC normally provides DNS for these domains through its internal DNS service. ([CRC.dev][1])

---

# 30. Flush Windows DNS cache

After modifying:

```text
C:\Windows\System32\drivers\etc\hosts
```

run PowerShell/CMD as Administrator:

```powershell
ipconfig /flushdns
```

Then test:

```powershell
nslookup console-openshift-console.apps-crc.testing
```

and:

```powershell
nslookup oauth-openshift.apps-crc.testing
```

The desired destination is:

```text
192.168.1.2
```

---

# 31. Final access URL

The OpenShift console is:

```text
https://console-openshift-console.apps-crc.testing
```

The browser path is:

```text
Windows
   |
   | HTTPS
   | console-openshift-console.apps-crc.testing
   |
   | hosts file
   ↓
192.168.1.2
   |
   | RHEL
   | HAProxy :443
   ↓
192.168.130.11
   |
   | CRC
   ↓
OpenShift Console
```

---

# 32. Final network architecture

Your finished environment looks like this:

```text
                         LOCAL LAN
                    192.168.1.0/24
                           |
                           |
                    +------v------+
                    |   WINDOWS   |
                    |             |
                    | hosts file  |
                    +------+------+
                           |
                           | 192.168.1.2
                           |
                    +------v------+
                    |    RHEL     |
                    |  SERVER     |
                    |             |
                    |  enp1s0    |
                    | 192.168.1.2|
                    |             |
                    |  HAProxy    |
                    |  :80        |
                    |  :443       |
                    |  :6443     |
                    +------+------+
                           |
                           | CRC internal network
                           |
                    +------v------+
                    |  CRC VM     |
                    |             |
                    |192.168.130.11
                    |             |
                    | OpenShift   |
                    | 4.22.1      |
                    +-------------+
```

---

# 33. Important IP addresses

Keep these straight:

| Purpose                      | Address               |
| ---------------------------- | --------------------- |
| RHEL server LAN IP           | `192.168.1.2`         |
| RHEL gateway                 | `192.168.1.1`         |
| CRC VM internal IP           | `192.168.130.11`      |
| Windows connects to          | `192.168.1.2`         |
| CRC API internally           | `192.168.130.11:6443` |
| OpenShift HTTPS through RHEL | `192.168.1.2:443`     |
| OpenShift HTTP through RHEL  | `192.168.1.2:80`      |
| OpenShift API through RHEL   | `192.168.1.2:6443`    |

**Do not put `192.168.130.11` in the Windows hosts file.**

---

# 34. Current CRC health checks

Useful commands:

### CRC status

```bash
crc status
```

Expected:

```text
CRC VM:          Running
OpenShift:       Running (v4.22.1)
```

### CRC IP

```bash
crc ip
```

Expected:

```text
192.168.130.11
```

### Libvirt

```bash
sudo virsh domstate crc
```

Expected:

```text
running
```

### HAProxy

```bash
sudo systemctl status haproxy --no-pager
```

### Firewall

```bash
sudo firewall-cmd --list-all
```

### Listening ports

```bash
sudo ss -lntp | grep -E ':(80|443|6443)\b'
```

---

# 35. Accessing `oc`

Earlier, you got:

```text
-bash: oc: command not found
```

That's because `oc` wasn't in your normal `$PATH`.

CRC provides the bundled `oc` environment. Run:

```bash
eval $(crc oc-env)
```

Then:

```bash
oc version
```

and:

```bash
oc whoami
```

For a normal CRC environment, you can obtain the console credentials with:

```bash
crc console --credentials
```

CRC documentation also uses `crc oc-env` to make the bundled OpenShift CLI available. ([CRC.dev][3])

---

# 36. Useful CRC commands

### Start

```bash
crc start
```

### Stop

```bash
crc stop
```

### Status

```bash
crc status
```

### IP

```bash
crc ip
```

### Console URL

```bash
crc console --url
```

### Credentials

```bash
crc console --credentials
```

### Configure `oc`

```bash
eval $(crc oc-env)
```

### CRC configuration

```bash
crc config view
```

### Network mode

```bash
crc config get network-mode
```

Expected:

```text
network-mode : system
```

---

# 37. Important disk-maintenance information

Your RHEL server originally had:

```text
Physical disk: 80G
```

but only:

```text
~39G
```

was allocated to the LVM partition.

We expanded it to:

```text
/dev/vda2 ≈ 79G
```

and:

```text
rhel-root ≈ 75G
```

with:

```text
swap ≈ 4G
```

Final root:

```text
/dev/mapper/rhel-root  75G
```

CRC's cache occupies approximately:

```text
34G
```

So keep an eye on:

```bash
df -h /
```

and:

```bash
du -sh /home/admin/.crc
du -sh /home/admin/.crc/cache
```

If the root filesystem approaches 100% again, CRC can pause because its VM disk is stored under `/home/admin/.crc`.

---

# 38. Windows hosts file — current version

At minimum, you currently need:

```text
192.168.1.2 console-openshift-console.apps-crc.testing
192.168.1.2 oauth-openshift.apps-crc.testing
192.168.1.2 api.crc.testing
```

This is fine for the console/login flow.

However, **hosts-file entries are not wildcard entries**. If you deploy applications with arbitrary OpenShift routes such as:

```text
myapp-myproject.apps-crc.testing
```

you would need DNS that handles the entire `apps-crc.testing` domain rather than manually adding every route.

CRC's official remote-client documentation recommends DNS/dnsmasq for remote clients for exactly this reason. ([CRC.dev][1])

---

# 39. Security warning

This setup is intended for your **local LAN**.

Do **not** port-forward these ports from your Internet router:

```text
80
443
6443
```

directly to the RHEL machine.

CRC's official documentation explicitly says the remote-server procedure should only be performed on a local network because exposing the server to the Internet has security implications. ([CRC.dev][1])

---

# 40. Final checklist

Your current setup should satisfy:

```text
[✓] RHEL server installed
[✓] RHEL server IP = 192.168.1.2
[✓] 80 GB disk detected
[✓] /dev/vda2 expanded to ~79 GB
[✓] LVM PV resized
[✓] root LV expanded to ~75 GB
[✓] XFS filesystem expanded
[✓] CRC 2.62.0 installed
[✓] OpenShift 4.22.1 bundle installed
[✓] Red Hat pull secret configured
[✓] libvirt working
[✓] KVM working
[✓] CRC VM running
[✓] CRC system network mode enabled
[✓] CRC VM IP = 192.168.130.11
[✓] CRC OpenShift = Running
[✓] CRC API reachable on 192.168.130.11:6443
[✓] HAProxy installed
[✓] HAProxy configured
[✓] SELinux configured for 6443
[✓] firewalld configured
[✓] HAProxy exposing 80/443/6443
[✓] Windows hosts file configured
[✓] console hostname configured
[✓] oauth hostname configured
[✓] api hostname configured
```

### Official reference

The authoritative CRC documentation for the remote-server portion is [CRC Networking / Remote Server documentation](https://crc.dev/docs/networking/?utm_source=chatgpt.com). It documents the system-networking requirement, HAProxy configuration, firewall ports, SELinux configuration, and remote DNS setup. ([CRC.dev][1])

**One thing I'd recommend next:** once the console is working, set up **proper DNS for `*.apps-crc.testing` on your LAN instead of continuing to add individual Windows hosts-file entries**. That will make every OpenShift Route you create automatically accessible from Windows.

[1]: https://crc.dev/docs/networking/?utm_source=chatgpt.com "Networking :: CRC Documentation"
[2]: https://crc.dev/docs/installing/?utm_source=chatgpt.com "Installing CRC :: CRC Documentation"
[3]: https://crc.dev/engineering-docs/Adding-an-insecure-registry.html?utm_source=chatgpt.com "Add an insecure registry - CRC Engineering docs"
