## Project 3: Web Server in a Namespace

### 1. (Optional) Recreate Project 1 Network

```sh
sudo ip netns add pod1
sudo ip netns add pod2

sudo ip link add veth-pod1 type veth peer name veth-pod2

sudo ip link set veth-pod1 netns pod1
sudo ip link set veth-pod2 netns pod2

sudo ip netns exec pod1 ip addr add 10.1.1.1/24 dev veth-pod1
sudo ip netns exec pod2 ip addr add 10.1.1.2/24 dev veth-pod2

sudo ip netns exec pod1 ip link set lo up
sudo ip netns exec pod1 ip link set veth-pod1 up
sudo ip netns exec pod2 ip link set lo up
sudo ip netns exec pod2 ip link set veth-pod2 up
```

---

### 2. Start Simple Web Server in pod1

```sh
sudo ip netns exec pod1 bash -c "echo 'Hello from Pod1 Web Server' > index.html && python3 -m http.server 8080 --bind 10.1.1.1" &
```

---

### 3. Curl the Web Page from pod2

```sh
sudo ip netns exec pod2 curl http://10.1.1.1:8080
```

**Expected output:**
```
Hello from Pod1 Web Server
```

---

### 4. Clean Up

```sh
sudo pkill -f "python3 -m http.server"
sudo ip netns del pod1
sudo ip netns del pod2
```

---
