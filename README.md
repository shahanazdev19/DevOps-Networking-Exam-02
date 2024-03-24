# DevOps-Networking-Exam-02
[ egress traffic ]
Connecting a container network namespace to root network namespace
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
First I setup the environment to get the basic commands in ubunto 20.4
Commands are-
  sudo apt-get update
  sudo apt-get install iptables iproute2 net-tools iputils-ping
Then the following steps I continued---
  1. Created a namespace in the server and a bridge- ns0 and br0
       sudo ip netns add ns0
       sudo ip link add br0 type bridge
  3. Changed the state of both namespace and bridge from DOWN to UP
       sudo ip link set br0 up
       sudo ip addr add 192.168.0.1/16 dev br0
  5. Created a vertual ethernet named veth0 with peer ceth0
       sudo ip link add veth0 type veth peer name ceth0 
  7. Connected the namespace ns0 to the vertual ethernet ceth0 and on ther other side of vertual ethernet to the bridge br0
       sudo ip link set ceth0 netns ns0
       sudo ip link set veth0 master br0
       sudo ip netns exec ns0 ip link set ceth0 up
       sudo ip link set veth0 up
  9. Then assigned ip address to both namesapce side veth (ceth0) and to bridge (br0)
       sudo ip link set lo up
       sudo ip addr add 192.168.0.2/16 dev ceth0
  11. Check the current ip addres of the server
        ip addr
  13.  Assigned a default gateway in the namespace ns0 to connect to the outher server from the assigned subnet mask        
        ip netns exec ns0 bash
        ip route add default via 192.168.0.1
  15. Connect to the server from namespace ns0
        sudo ip netns exec ns0 bash
        ping 10.0.0.25 -c 5

These are the steps to connect a namespace to a server internally.
Now next task is to connect the namespace to the outer world, so that any outside server can get connected to the namespace
For this I applied SNAT Rule at Host Server, not in namespace
  sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16  -j MASQUERADE

Then applied some firewall rule.
  To verify iptables chain and other stuff
      sudo iptables -t nat -L -n -v
  In case if still it not works then we may need to add some additional firewall rules.
      sudo iptables --append FORWARD --in-interface br0 --jump ACCEPT
      sudo iptables --append FORWARD --out-interface br0 --jump ACCEPT

Now the server and the namespace are ready to connect to any outer world server.
  Test connectivity from server:
      ping 8.8.8.8 -c 5
  Test connectivity from namespace inside server:
      sudo ip netns exec ns0 bash
      ping 8.8.8.8 -c 5

  N.B: To exit from namespace to main server press ctr+C and write exit
  
