# Goal: Two pods(namespaces) talking to each other directly

<img width="282" height="152" alt="image" src="https://github.com/user-attachments/assets/d56ee3d7-45a8-4d30-98e1-5baef783c040" />


# Create namespaces
sudo ip netns add pod1
sudo ip netns add pod2

# Create veth pair (virtual cable)
sudo ip link add veth-pod1 type veth peer name veth-pod2
sudo ip link set veth-pod1 netns pod1
sudo ip link set veth-pod2 netns pod2

# Assign IPs
sudo ip netns exec pod1 ip addr add 10.1.1.1/24 dev veth-pod1
sudo ip netns exec pod2 ip addr add 10.1.1.2/24 dev veth-pod2

# Bring interfaces up
sudo ip netns exec pod1 ip link set lo up
sudo ip netns exec pod1 ip link set veth-pod1 up
sudo ip netns exec pod2 ip link set lo up
sudo ip netns exec pod2 ip link set veth-pod2 up

# Test
sudo ip netns exec pod1 ping -c 3 10.1.1.2
