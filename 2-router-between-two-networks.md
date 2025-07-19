# Goal: Two separate networks talk to each other via a router namespace.

### 1. Clean Up (Optional)

```sh
for ns in pod1 pod2 rtr; do sudo ip netns del $ns 2>/dev/null; done
sudo ip link del veth-pod1 2>/dev/null || true
sudo ip link del veth-pod2 2>/dev/null || true
```

---

### 2. Create Namespaces

```sh
sudo ip netns add pod1
sudo ip netns add pod2
sudo ip netns add rtr
```

---

### 3. Create veth Pairs and Attach to Namespaces

```sh
sudo ip link add veth-pod1 type veth peer name veth-rtr1
sudo ip link add veth-pod2 type veth peer name veth-rtr2

sudo ip link set veth-pod1 netns pod1
sudo ip link set veth-rtr1 netns rtr

sudo ip link set veth-pod2 netns pod2
sudo ip link set veth-rtr2 netns rtr
```

---

### 4. Assign IP Addresses

```sh
sudo ip netns exec pod1 ip addr add 10.2.1.2/24 dev veth-pod1
sudo ip netns exec rtr  ip addr add 10.2.1.1/24 dev veth-rtr1

sudo ip netns exec pod2 ip addr add 10.2.2.2/24 dev veth-pod2
sudo ip netns exec rtr  ip addr add 10.2.2.1/24 dev veth-rtr2
```

---

### 5. Bring Interfaces Up

```sh
for ns in pod1 pod2 rtr; do sudo ip netns exec $ns ip link set lo up; done

sudo ip netns exec pod1 ip link set veth-pod1 up
sudo ip netns exec rtr  ip link set veth-rtr1 up

sudo ip netns exec pod2 ip link set veth-pod2 up
sudo ip netns exec rtr  ip link set veth-rtr2 up
```

---

### 6. Enable Routing in Router

```sh
sudo ip netns exec rtr sysctl -w net.ipv4.ip_forward=1
```

---

### 7. Add Default Routes in Pods

```sh
sudo ip netns exec pod1 ip route add default via 10.2.1.1
sudo ip netns exec pod2 ip route add default via 10.2.2.1
```

---

### 8. Test Connectivity (Pod1 to Pod2 via Router)

```sh
sudo ip netns exec pod1 ping -c 3 10.2.2.2
```

---

