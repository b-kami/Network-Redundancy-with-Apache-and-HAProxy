# Network Redundancy with Apache and HAProxy

This project demonstrates the configuration of a highly available and redundant network using Apache web servers and HAProxy load balancers. It guides you through setting up redundancy and testing failover mechanisms to ensure seamless service availability.

## **Overview**

The project involves:
1. Configuring two Apache web servers with primary-backup redundancy.
2. Setting up two HAProxy load balancers with active-passive failover.
3. Using Keepalived to manage the failover for HAProxy using a virtual IP (VIP).
4. Testing the setup by simulating failures and observing automatic failover behavior.

---

## **Requirements**

1. **Environment**:
   - 5 Ubuntu virtual machines:
     - 2 Apache web servers
     - 2 HAProxy load balancers
     - 1 client machine for testing
   - Networking setup via tools like GNS3 or VMware to allow communication between VMs.

2. **Software**:
   - Apache (`apache2`)
   - HAProxy
   - Keepalived

3. **Basic Knowledge**:
   - Apache server configuration
   - Load balancing with HAProxy
   - Keepalived for high availability

---

## **Network Topology**

![Network Topology](path/to/topology-image.png)

- **Apache Servers**:
  - Server Web 1: `192.168.2.13`
  - Server Web 2: `192.168.2.14`
- **HAProxy Load Balancers**:
  - HAProxy 1: `192.168.2.11`
  - HAProxy 2: `192.168.2.12`
- **Virtual IP (VIP)**: `192.168.2.100`

---

## **Setup Steps**

### **1. Configure Apache Web Servers**
1. **Install Apache**:
   ```bash
   sudo apt update
   sudo apt install apache2 -y
   ```

2. **Set Up Web Pages**:
   - On **Server Web 1**:
     ```bash
     echo "Server Web 1 - Primary" | sudo tee /var/www/html/index.html
     ```
   - On **Server Web 2**:
     ```bash
     echo "Server Web 2 - Backup" | sudo tee /var/www/html/index.html
     ```

3. **Test Apache**:
   - Access the pages via:
     - `http://192.168.2.13` (Server Web 1)
     - `http://192.168.2.14` (Server Web 2)

---

### **2. Configure HAProxy Load Balancers**
1. **Install HAProxy**:
   ```bash
   sudo apt update
   sudo apt install haproxy -y
   ```

2. **Configure HAProxy**:
   - Edit `/etc/haproxy/haproxy.cfg` on both HAProxy servers:
     ```plaintext
     frontend http_front
         bind *:80
         default_backend web_servers

     backend web_servers
         balance roundrobin
         server web1 192.168.2.13:80 check
         server web2 192.168.2.14:80 check
     ```

3. **Restart HAProxy**:
   ```bash
   sudo systemctl restart haproxy
   ```

4. **Test Load Balancing**:
   - Access HAProxy via `http://192.168.2.11` to verify traffic distribution.

---

### **3. Set Up Keepalived for HAProxy Failover**
1. **Install Keepalived**:
   ```bash
   sudo apt install keepalived -y
   ```

2. **Configure Keepalived**:
   - On **HAProxy 1** (`/etc/keepalived/keepalived.conf`):
     ```plaintext
     vrrp_instance VI_1 {
         state MASTER
         interface eth0
         virtual_router_id 51
         priority 101
         advert_int 1
         authentication {
             auth_type PASS
             auth_pass securepass
         }
         virtual_ipaddress {
             192.168.2.100
         }
         track_script {
             chk_haproxy
         }
     }
     ```

   - On **HAProxy 2**:
     - Use the same configuration but set `state` to `BACKUP` and `priority` to `100`.

3. **Restart Keepalived**:
   ```bash
   sudo systemctl start keepalived
   ```

4. **Verify VIP**:
   - Run `ip addr` on both HAProxy servers to confirm the VIP (`192.168.2.100`) is active on the primary server.

---

### **4. Test Failover and Redundancy**
1. **Test Load Balancing**:
   - Access `http://192.168.2.100` to confirm balanced traffic between Apache servers.

2. **Simulate Failures**:
   - Stop Apache on Server Web 1:
     ```bash
     sudo systemctl stop apache2
     ```
   - Verify traffic redirects to Server Web 2.

   - Stop HAProxy on HAProxy 1:
     ```bash
     sudo systemctl stop haproxy
     ```
   - Confirm the VIP transfers to HAProxy 2.

3. **Document Results**:
   - Note failover times and check for service continuity.

---

## **Expected Results**

1. **Load Balancing**: Traffic is distributed evenly between Server Web 1 and Server Web 2.
2. **Failover**: In case of failures, backup servers seamlessly take over with minimal downtime.

---

## **Conclusion**

This project demonstrates the effectiveness of redundancy and high availability in network configurations. The use of Apache, HAProxy, and Keepalived ensures uninterrupted service, even during system failures. Future improvements could include advanced monitoring or scaling mechanisms.

---

Feel free to ask if you need help with GitHub commands or anything else!
