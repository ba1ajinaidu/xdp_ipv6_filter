# XDP based IPv6 Packet filter

### Prerequisites
- Latest kernel with XDP enabled on it. This program was developed and tested on `Ubuntu 22.04` OS and `Linux 5.15.0-46-generic` kernel.
- iproute2

### How to run

1. Compile the program. This creates `xdp_ipv6_filter.o` object file.
   ```
   $ make
   clang -I. -I/lib/modules/5.15.0-46-generic/build/arch/x86/include -I/lib/modules/5.15.0-46-generic/build/arch/x86/include/generated - I/lib/modules/5.15.0-46-generic/build/include -I/lib/modules/5.15.0-46-generic/build/arch/x86/include/uapi -I/lib/modules/5.15.0-46-generic/build/include/uapi -include /lib/modules/5.15.0-46-generic/build/include/linux/kconfig.h -D__KERNEL__ -D__BPF_TRACING__ -Wno-unused-value -Wno-pointer-sign -D__TARGET_ARCH_x86 -Wno-compare-distinct-pointer-types -Wno-gnu-variable-sized-type-not-at-end -Wno-address-of-packed-member -Wno-tautological-compare -O2 -emit-llvm -c xdp_ipv6_filter.c -o - | \
   llc -march=bpf -mcpu= -filetype=obj -o xdp_ipv6_filter.o
    ```
2. Create a veth pair using iproute.
   ```
   $ sudo ip link add dev veth0 type veth peer name veth1
   $ sudo ip link set up dev veth0
   $ sudo ip link set up dev veth1
   ```
3. Attach the compiled XDP program(`xdp_ipv6_filter.o`) to veth0.
   ```
   sudo ip link set dev veth0 xdp object xdp_ipv6_filter.o
   ```
4. Run tcpdump on veth0 to monitor and capture the IPv6 packets.
   ```
   sudo tcpdump "ip6" -i veth0 -w captured.pcap -c 10
   ```
5. To generate ipv6 packets we ping veth0 via veth1. Grab the IPv6 address of `veth0` interface using `ip a` command and ping the interface.
   ```
   ping6 fe80::ec53:40ff:fe29:5d68%veth1 -c5
   ```   
