Basic Not final
```
openshift-admin1-do180/
│
├── README.md
│
├── 00-foundations/
│   ├── 01-linux-basics.md
│   ├── 02-ip-addressing.md
│   ├── 03-subnets-and-cidr.md
│   ├── 04-routing-and-gateways.md
│   ├── 05-dns.md
│   ├── 06-tcp-udp-and-ports.md
│   ├── 07-http-https.md
│   ├── 08-firewalls.md
│   ├── 09-load-balancers.md
│   └── 10-network-troubleshooting.md
│
├── 01-containers/
│   ├── what-is-a-container.md
│   ├── images.md
│   ├── registries.md
│   ├── podman-basics.md
│   ├── container-networking.md
│   └── dockerfile.md
│
├── 02-kubernetes/
│   ├── architecture.md
│   ├── pods.md
│   ├── deployments.md
│   ├── namespaces.md
│   ├── services.md
│   ├── configmaps.md
│   ├── secrets.md
│   └── kubernetes-networking.md
│
├── 03-openshift/
│   ├── what-is-openshift.md
│   ├── kubernetes-vs-openshift.md
│   ├── openshift-architecture.md
│   ├── projects.md
│   └── cluster-components.md
│
├── 04-cluster-setup/
│   ├── crc-installation.md
│   ├── crc-setup-steps.md
│   ├── production-architecture.md
│   └── crc-troubleshooting.md
│
├── 05-cli-basics/
│   ├── oc-login.md
│   ├── oc-projects.md
│   ├── oc-basic-commands.md
│   └── oc-output-formats.md
│
├── 06-application-deployment/
│   ├── deploy-from-image.md
│   ├── deploy-from-yaml.md
│   ├── deployments.md
│   ├── templates.md
│   └── image-streams.md
│
├── 07-networking/
│   ├── pod-networking.md
│   ├── services.md
│   ├── routes.md
│   ├── ingress.md
│   ├── network-policies.md
│   └── troubleshooting.md
│
├── 08-storage/
│   ├── volumes.md
│   ├── pv-pvc.md
│   ├── storage-classes.md
│   └── dynamic-provisioning.md
│
├── 09-scaling-high-availability/
│   ├── replicas.md
│   ├── deployments.md
│   ├── autoscaling.md
│   ├── health-checks.md
│   └── pod-disruption-budgets.md
│
├── 10-security/
│   ├── users-roles.md
│   ├── rbac.md
│   ├── service-accounts.md
│   ├── scc.md
│   ├── secrets.md
│   └── network-policies.md
│
├── 11-monitoring-troubleshooting/
│   ├── logs.md
│   ├── debugging-pods.md
│   ├── events.md
│   ├── resource-usage.md
│   └── troubleshooting-methodology.md
│
├── 12-production/
│   ├── production-architecture.md
│   ├── networking-design.md
│   ├── dns-design.md
│   ├── load-balancing.md
│   ├── certificates.md
│   ├── security-hardening.md
│   ├── backup-and-recovery.md
│   ├── monitoring.md
│   └── disaster-recovery.md
│
├── labs/
│   ├── 01-ip-networking.md
│   ├── 02-dns.md
│   ├── 03-containers.md
│   ├── 04-first-openshift-app.md
│   ├── 05-services-and-routes.md
│   ├── 06-storage.md
│   ├── 07-scaling.md
│   ├── 08-security.md
│   └── 09-troubleshooting.md
│
├── scripts/
│   ├── crc-setup.sh
│   └── oc-basic-setup.sh
│
└── images/
    ├── network-basics.png
    ├── openshift-architecture.png
    └── pod-flow.png
```
Structure
```
PHYSICAL SERVER (RHEL 9.7)
   ├── CPU: Intel Xeon (VT-x enabled)
   ├── RAM: 660 GB
   ├── KERNEL: KVM enabled (kvm + kvm_intel)
   │
   ├── libvirt (virsh management layer)
   │
   ├── QEMU/KVM virtual machines
   │     ├── VM(s)
   │     │     └── Docker inside VM
   │     │           ├── go-backend (8080)
   │     │           └── react-frontend
```
