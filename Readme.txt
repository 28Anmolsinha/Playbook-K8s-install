# Kubernetes Installation Using Ansible Playbook

This guide provides a step-by-step approach to installing Kubernetes using an Ansible playbook. Before running the playbook, ensure that all pre-existing Kubernetes configurations and dependencies are removed to avoid conflicts.

---

## 🛠 Checking for an Existing Kubernetes Cluster

To verify if Kubernetes is installed, run:

```bash
kubectl cluster-info
```

### Possible Outputs:
1. **If Kubernetes is running:**
   ```
   Kubernetes control plane is running at https://<master-ip>:6443
   CoreDNS is running at https://<master-ip>/api/v1/namespaces/kube-system/services/kube-dns
   ```
2. **If Kubernetes is not installed:**
   ```
   The connection to the server localhost:8080 was refused - did you specify the right host or port?
   ```
   This indicates that Kubernetes is either not running or `kubectl` is misconfigured.

---

## 🔥 Removing an Existing Kubernetes Cluster

If a Kubernetes cluster exists, remove it using the following steps:

1. **Reset the cluster:**
   ```bash
   kubeadm reset
   ```
2. **Uninstall Kubernetes components:**
   ```bash
   sudo yum remove kubeadm kubectl kubelet kubernetes-cni kube*
   ```
3. **Remove unnecessary dependencies:**
   ```bash
   sudo yum autoremove
   ```
4. **Delete Kubernetes configuration files:**
   ```bash
   sudo rm -rf ~/.kube
   ```

### 🖥 Checking Active Nodes

To list active nodes in an existing cluster:
```bash
kubectl get nodes
```
- If a cluster exists, it will display active nodes.
- If no cluster exists, an error message will appear.

### ❌ Removing a Node (If Exists)

1. Drain the node to move workloads:
   ```bash
   kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
   ```
2. Delete the node from the cluster:
   ```bash
   kubectl delete node <node-name>
   ```
3. Reset Kubernetes on the node:
   ```bash
   kubeadm reset
   ```

### 🗑 Removing Pre-Existing Kubernetes Directories

Ensure a clean environment by deleting the following directories:
```bash
rm -rf ~/.kube /etc/kubernetes /var/lib/kubelet /var/lib/etcd /var/lib/cni
```

Once completed, Kubernetes and its configurations will be fully removed.

---

# 🚀 Configuring Ansible Playbook for Kubernetes Installation

## 📂 Step 1: Navigate to the Playbook Directory
```bash
cd K8s-install
```
The primary playbook file **playbook.yml** will be executed for Kubernetes installation.

## 📝 Step 2: Modify the Ansible Playbook
1. Open the playbook file:
   ```bash
   vim playbook.yml
   ```
2. Press `i` to enter insert mode.
3. Modify the **hosts** section to define target machines.

## 🏗 Step 3: Updating the Inventory File

1. Open the inventory file:
   ```bash
   vim inventory.ini
   ```
2. List the servers where you want to execute the Ansible playbook.
3. Ensure that the group name in `playbook.yml` matches the one in `inventory.ini`.

## ⚙️ Step 4: Preparing the System for Installation

Ensure the target system does not have any pre-installed Kubernetes or `containerd` packages.

🔹 It is recommended to execute the playbook from a directory with sufficient storage space.

---

## 📌 Playbook Structure

The playbook consists of the following roles:

### 1️⃣ `pretask` - Pre-Installation Configuration
- **Disables Swap** (required by Kubernetes for stability)
- **Disables Firewall** (prevents interference with Kubernetes networking)
- **Configures SELinux** (sets it to permissive mode)
- **Configures Netfilter (br_netfilter)** (enables packet forwarding & `iptables` compatibility)

### 2️⃣ `containerd` - Install and Configure Container Runtime
- **Installs containerd**
- **Configures SystemdCgroup**
- **Restarts containerd service**

### 3️⃣ `instkube` - Install and Configure Kubernetes Components
- Installs `kubeadm`, `kubelet`, and `kubectl`
- Initializes Kubernetes cluster (`kubeadm init`)
- Downloads and applies **Calico** networking plugin

### 4️⃣ `insthelm` - Install and Configure Helm
- Downloads and installs Helm
- Moves Helm binary to appropriate system directory

### 📂 `main.yml` (Predefined Task Entry Point)
- Ensures tasks within each role are executed in an organized manner.

---

## 🏗 Executing the Playbook

Run the playbook using:
```bash
sudo ansible-playbook playbook.yml
```

### 🔹 Running with an Inventory File
```bash
sudo ansible-playbook -i inventory.ini playbook.yml
```

This command executes `playbook.yml`, which triggers all roles and defined tasks. The output logs will help debug any errors encountered during execution.

---

## 🎯 Summary
This guide provides a structured method to remove any pre-existing Kubernetes installations and deploy a fresh Kubernetes cluster using an Ansible playbook. By following the defined roles and configurations, you ensure a smooth and automated setup of Kubernetes with all necessary components.

🚀 Happy Kubernetes Deployment! 🚀

