
## Fedora CoreOS

Fedora CoreOS (FCOS) is a minimal operating system designed for running containerized workloads securely, at scale. 
This operating system building blocks are the great CoreOS and Fedora Atomic. 
It has a feature of automated updates and is immutable to ensure the OS is stable and reliable. 
The OS automatically updates itself with the latest OS improvements, bug fixes, and security updates with rpm-ostree.

![image](https://user-images.githubusercontent.com/3519706/77247103-db39ec80-6c3e-11ea-9a8d-d2a00b8d7985.png)

Unlike other Linux operating systems, Fedora CoreOS (FCOS) has no install-time configuration. 
Every FCOS system begins with a generic disk image. For each deployment mechanism (cloud VM, local VM, bare metal), configuration can be supplied at first boot. 
FCOS reads and applies the configuration file with Ignition.

When doing Fedora CoreOS installation on bare metal, or as a Virtual Machine with an ISO file, the Ignition will inject the configuration at install time. 
But for the deployments being done in a cloud environment, Ignition will gather the configuration via the cloud’s user-data mechanism.

We made 2 scenarios within the scope of Fedora CoreOS

Fedora CoreOS Install - [Kubernetes_Install_on_Fedora_CoreOS.md](https://github.com/OktaySavdi/CoreOS/blob/master/Kubernetes_Install_on_Fedora_CoreOS.md "Kubernetes_Install_on_Fedora_CoreOS.md")

Kubernetes install on CoreOS - [install_Fedora_CoreOS.md](https://github.com/OktaySavdi/CoreOS/blob/master/install_Fedora_CoreOS.md "install_Fedora_CoreOS.md")
