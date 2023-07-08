# Kubernetes on RISC-V

As of mid-2023, Debian and Ubuntu Linux run on RISC-V, with container support enabled in the kernel by default, however lack Kubernetes binaries. [Previous work](https://github.com/carlosedp/riscv-bringup) in the direction of running containers and Kubernetes on RISC-V indicates that it is possible. Back then, Golang support for RISC-V was at “experimental” status; now RISC-V is fully supported, so cross-compiling to the new architecture should be easier.

This document marks our current progress in making Kubernetes available for RISC-V.

## Running

In summary, the following commands should get you up and running:
```bash
# Download and install K3s
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230616/k3s-riscv64.gz.aa
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230616/k3s-riscv64.gz.ab
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230616/k3s-riscv64.gz.ac
cat k3s-riscv64.gz.* | gunzip > /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s

curl -sfL https://get.k3s.io > k3s-install.sh
INSTALL_K3S_EXEC="server --disable traefik,metrics-server --pause-image carvicsforth/pause-riscv:v3.9-v1.27.2" INSTALL_K3S_SKIP_DOWNLOAD="true" bash -x k3s-install.sh

# Set the CoreDNS container image to "carvicsforth/coredns:1.10.1"
KUBE_EDITOR="sed -i '/image\:/ s/: .*/: carvicsforth\/coredns:1.10.1/'" \
kubectl edit deployment -n kube-system coredns

# Set the Local Path Provisioner container image to "carvicsforth/local-path-provisioner:master-head" and the helper pod container image to "riscv64/busybox:1.34.1"
KUBE_EDITOR="sed -i '/image\:/ s/: .*/: riscv64\/busybox:1.34.1/'" \
kubectl edit configmap -n kube-system local-path-config
KUBE_EDITOR="sed -i '/image\:/ s/: .*/: carvicsforth\/local-path-provisioner:master-head/'" \
kubectl edit deployment -n kube-system local-path-provisioner
```

> **Note**
> Most container images are not available for RISC-V, so many applications may not run.

To deploy a sample Ubuntu 22.04 pod:
```bash
kubectl apply -f examples/ubuntu-pod.yaml
```

## K3s

We started by cross-compiling [K3s](https://k3s.io/) to RISC-V. K3s is a light-weight Kubernetes distribution that packs all necessary code into a single binary and needs a smaller memory footprint to run.

In addition to cross-compiling the K3s core, respective changes where required in dependencies [k3s-root](https://github.com/k3s-io/k3s-root) (the base user space binaries for K3s) and [runc](https://github.com/opencontainers/runc) (the tool that runs the containers).

Here is a list of submitted PRs:
- [Add support for RISC-V](https://github.com/k3s-io/k3s/pull/7778) in k3s
- [Support RISC-V](https://github.com/k3s-io/k3s-root/pull/60) in k3s-root - *Merged, released in v0.13.0*
- [Backport riscv64 support into 1.1.x](https://github.com/opencontainers/runc/pull/3905) in runc - *Merged, not yet released*

Until RISC-V support is merged upstream in K3s, we maintain a [fork](https://github.com/CARV-ICS-FORTH/k3s) with [binaries and install instructions](https://github.com/CARV-ICS-FORTH/k3s/releases).

## Supporting services and container images

To support the effort, some services have also been ported to RISC-V:
- [Add support for RISC-V](https://github.com/coredns/coredns/pull/6195) in CoreDNS
- [Add support for RISC-V](https://github.com/rancher/local-path-provisioner/pull/346) in Local Path Provisioner

To build and upload the CoreDNS image from our fork, we comment out building `amd64` and `arm64` binaries in `Makefile.release` and run:
```bash
make -f Makefile.release build
mkdir -p build/docker/riscv64
cp build/linux/riscv64/coredns build/docker/riscv64/
make DOCKER=carvicsforth VERSION=1.10.1 -f Makefile.docker docker-build
docker tag docker.io/carvicsforth/coredns:riscv64-1.10.1 docker.io/carvicsforth/coredns:1.10.1
docker push carvicsforth/coredns:1.10.1
```

To build and upload the Local Path Provisioner image from our fork, we run:
```bash
ARCH=riscv64 SKIP_TEST=1 SKIP_VALIDATE=1 make ci
docker tag rancher/local-path-provisioner:7e1d2a1-riscv64 carvicsforth/local-path-provisioner:master-head
docker push carvicsforth/local-path-provisioner:master-head
```

Any other necessary container images (like the `pause` container) can be found in the `images` folder.

## Acknowledgements

This project has received funding from the European Union’s Horizon Europe research and innovation programme through project RISER ("RISC-V for Cloud Services", GA-101092993) and from the Key Digital Technologies Joint Undertaking through project REBECCA ("Reconfigurable Heterogeneous Highly Parallel Processing Platform for safe and secure AI", GA-101097224). KDT JU projects are jointly funded by the European Commission and the involved state members (including the Greek General Secretariat for Research and Innovation).
