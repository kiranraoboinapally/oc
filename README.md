Basic Not final
```
openshift-admin1-do180/
│
├── README.md
│
├── 00-introduction/
│   ├── what-is-openshift.md
│   ├── kubernetes-vs-openshift.md
│
├── 01-cluster-setup/
│   ├── crc-installation.md
│   ├── crc-setup-steps.md
│   ├── crc-troubleshooting.md
│
├── 02-cli-basics/
│   ├── oc-login.md
│   ├── oc-projects.md
│   ├── oc-basic-commands.md
│
├── 03-application-deployment/
│   ├── deploy-from-image.md
│   ├── deploy-from-yaml.md
│   ├── templates.md
│
├── 04-networking/
│   ├── services.md
│   ├── routes.md
│   ├── ingress.md
│
├── 05-storage/
│   ├── pv-pvc.md
│   ├── storage-classes.md
│
├── 06-scaling-high-availability/
│   ├── replicas.md
│   ├── autoscaling.md
│   ├── health-checks.md
│
├── 07-security-basics/
│   ├── users-roles.md
│   ├── scc.md
│
├── 08-monitoring-troubleshooting/
│   ├── logs.md
│   ├── debugging-pods.md
│   ├── events.md
│
├── 09-best-practices/
│   ├── performance.md
│   ├── security-practices.md
│
├── scripts/
│   ├── crc-setup.sh
│   ├── oc-basic-setup.sh
│
├── images/
│   ├── openshift-architecture.png
│   ├── pod-flow.png
│
└── labs/
    ├── lab-1-deploy-app.md
    ├── lab-2-networking.md
    ├── lab-3-storage.md
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
