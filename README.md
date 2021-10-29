# k8s_ovs_dpdk
The goal of this project is to enable east-west traffic for dpdk based applications running in pods on two different nodes.

Setup
VM0 - Nested Hypervisor [LINUX] + KVM + OVS-DPDK
VM1 [running on VM0] - K8s Master
VM2 [running on VM0] - K8s Worker0 // Enable DPDK based app in POD to send traffic to another app running on another POD [on worker1 node]
VM3 [running on VM0] - K8s Worker1


Milestones:
-----------
0. Install and configure VM0 with OVS-DPDK and KVM []
1. Figure out vhostuser
1. Install and configure VM1, enable east-west [over OVS] and north-south [over OVS?] traffic []
2. Install and configure VM2, enable east-west [over OVS] and north-south [over OVS?] traffic []
3. Install and configure VM3, enable east-west [over OVS] and north-south [over OVS?] traffic []
4. At this point we have running k8s cluster VM1, VM2 and VM3 connected with OVS-DPDK on VM0  []
5. TODO





Notes:
------

Install missing tools and libs:
sudo apt-get install meson python3-pyelftools python-pyelftools libz-dev libpcap-dev libnuma-dev libssl-dev autoconf libtool libdbus-1-dev git



DPDK Install
------------

git clone http://dpdk.org/git/dpdk \
cd dpdk \
meson build \
ninja -C build \
sudo ninja -C build install \
sudo ldconfig \
export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig/ \
// if version is printed all is ok \
pkg-config --modversion libdpdk \




OVS-DPDK install
----------------
// was unable to install from sources

sudo apt-get install openvswitch-switch-dpdk
sudo update-alternatives --set ovs-vswitchd /usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk
sudo ovs-vsctl set Open_vSwitch . "other_config:dpdk-init=true"

// run on core 0 only
sudo ovs-vsctl set Open_vSwitch . "other_config:dpdk-lcore-mask=0x1"

// run time allocation of huge pages where N = No. of 2M huge pages
sysctl -w vm.nr_hugepages=N

// verify hugepage configuration
grep HugePages_ /proc/meminfo

// mount the hugepages
sudo su -
mount -t hugetlbfs none /dev/hugepages``

// limit to one whitelisted device
sudo ovs-vsctl set Open_vSwitch . "other_config:dpdk-extra=--pci-whitelist=0000:04:00.0"

// restart ovs
sudo service openvswitch-switch restart
