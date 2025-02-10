#Kubernetes Installation Using Ansible Playbook  

This guide outlines the steps to install Kubernetes using an Ansible playbook. Before proceeding, ensure that all pre-existing Kubernetes configurations and dependencies are removed for a smooth setup.  

Before running the playbook, ensure the target system does not have any pre-installed Kubernetes or related  packages.  

#Checking for an Existing Kubernetes Cluster  

To verify if a Kubernetes cluster is installed, run:  

kubectl cluster-info  

### Possible Outputs:  
1. If Kubernetes is running, you will see:  
   Kubernetes control plane is running at https://<master-ip>:6443  
   CoreDNS is running at https://<master-ip>/api/v1/namespaces/kube-system/services/kube-dns  

2. If Kubernetes is not installed, you will see:  
   The connection to the server localhost:8080 was refused - did you specify the right host or port?  

This indicates that Kubernetes is either not running or `kubectl` is misconfigured.  

## Removing an Existing Kubernetes Cluster  

If a Kubernetes cluster exists, remove it using the following steps:  
1. Reset the cluster:  
   kubeadm reset  
2. Uninstall Kubernetes components:  
   sudo yum remove kubeadm kubectl kubelet kubernetes-cni kube*  
3. Remove unnecessary dependencies:  
   sudo yum autoremove  
4. Delete Kubernetes configuration files:  
   sudo rm -rf ~/.kube  

## Checking Active Nodes  

To list active nodes in an existing cluster:  
kubectl get nodes  

- If a cluster exists, it will display active nodes.  
- If no cluster exists, an error message will appear.  

## Removing a Node (If Exists)  

1. Drain the node to move workloads:  
   kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data  
2. Delete the node from the cluster:  
   kubectl delete node <node-name>  
3. Reset Kubernetes on the node:  
   kubeadm reset  

## Removing the Entire Kubernetes Cluster  

To completely remove all nodes and configurations:  
1.kubeadm reset  
2.rm -rf ~/.kube  

## Removing Pre-Existing Kubernetes Directories  

Ensure a clean environment by deleting the following directories:  
1. rm -rf ~/.kube  
2. rm -rf /etc/kubernetes  
3. rm -rf /var/lib/kubelet  
4. rm -rf /var/lib/etcd  
5. rm -rf /var/lib/cni  

Once all these steps are completed, Kubernetes and its related configurations will be fully removed from the system.  

# Configuring Ansible Playbook for Kubernetes Installation  

## Step 1: Navigate to the Playbook Directory  

cd K8s-install  

Inside this directory, the primary playbook file **playbook.yml** will be executed for Kubernetes installation.  

## Step 2: Modify the Ansible Playbook  

1. Open the playbook file for editing:  
   vim playbook.yml  
2. Press `i` to enter insert mode.  
3. Modify the **hosts** section to define where the commands will be executed.  

## Step 3: Updating the Inventory File  

- Open the inventory file:  
  vim inventory.ini  
- List the desired servers where you want to execute the Ansible playbook.  
- You can either:  
  - Add your server to an existing group, OR  
  - Create a new group for individual use.  

**Important:** Ensure that the group name in `playbook.yml` matches the one in `inventory.ini`, or the playbook execution will fail.  

## Step 4: Preparing the System for Installation  

Before running the playbook, ensure the target system does not have any pre-installed Kubernetes or `containerd` packages.  

Make sure to name the exact group name in the playbook.yml file in the host section else it won't get configure and will effect the execution of the playbook resuling in undesired output.

It is recommended to go inside the data directory or any such directory having the maximum space as it is required for easier execution.

## Playbook Structure  

The playbook includes the following roles:  

1. pretask – Ensures all necessary system prerequisites are met.  
2. containerd – Installs and configures `containerd` as the container runtime.  
3. instkube – Installs Kubernetes components (`kubeadm`, `kubelet`, and `kubectl`).  
4. insthelm – Installs Helm, a package manager for Kubernetes.  

--------

## Role1: pretask (Pre-Installation Configuration)  

The **pretask** role ensures the system is properly configured before installing Kubernetes. It performs the following tasks:  

-  Disabling Swap : Kubernetes requires swap to be disabled for stability and optimal performance.  
-  Disabling Firewall : Prevents firewall rules from interfering with Kubernetes networking and communication between nodes.  
-  Configuring SELinux : Ensures SELinux is set to permissive mode to avoid permission-related issues during Kubernetes operations. 
-  Configuring Netfilter (br_netfilter) :  
  - Enables packet forwarding and proper communication between pods.  
  - Ensures `iptables` sees bridged traffic, which is essential for Kubernetes networking

These configurations are critical to prevent potential issues during cluster setup and ensure Kubernetes functions correctly.  

----


## Role2: containerd (Install and Configure Container Runtime)  

The **containerd** role is responsible for setting up the container runtime required for Kubernetes. It consists of the following tasks:  

-  Install_containerd :
  - uninstall older docker version  
  - Installs the `containerd` runtime along with necessary dependencies.  
  - Ensures the latest version is fetched and properly set up for Kubernetes.  

-  configure_containerd :  
  - Modifies the default `containerd` configuration to optimize it for Kubernetes.  
  - Enables `SystemdCgroup` to ensure compatibility with Kubernetes cluster management.  
  - Restarts the `containerd` service to apply configuration changes.  

These configurations ensure that `containerd` functions as the primary container runtime for Kubernetes, providing a stable and efficient environment.  


----

## Role3: instkube (Install and Configure Kubernetes Components)  

The instkube role is responsible for setting up the core Kubernetes components required for cluster initialization and networking. It consists of the following tasks:  

-  Installk8.yml :  
  - Installs essential Kubernetes packages, including `kubeadm`, `kubelet`, and `kubectl`.  
  - Ensures that the correct versions are installed and configured for cluster setup.  

-  init_cluster.yml :  
  - Initializes the Kubernetes cluster on the master node using `kubeadm init`.  
  - Sets up the control plane and configures necessary components for cluster operation.  
  - Generates and applies the required cluster configuration files.  

- Download_calico.yml :  
  - Downloads and applies the Calico networking plugin for Kubernetes.  
  - Configures network policies and ensures proper pod-to-pod communication.  

These tasks collectively ensure the successful deployment of a Kubernetes cluster with a properly configured control plane and networking.  

-----


## Role4: insthelm (Install and Configure Helm)  

The insthelm  role is responsible for installing and setting up Helm, the Kubernetes package manager. It consists of the following tasks:  

-  install_helm.yml :  
  - Downloads the latest Helm binary from the official source.  
  - Installs Helm and ensures it is available system-wide for managing Kubernetes applications.  

-  extract_helm.yml :  
  - Extracts the Helm package and moves the binary to the appropriate system directory.  
  - Verifies Helm installation to ensure it is correctly set up and ready for use.  

These tasks ensure that Helm is properly installed and configured, allowing seamless deployment and management of applications within the Kubernetes cluster.  

----

##Main.yml ( A pre-defined task )

- The main.yml file is a crucial component found within each role in Ansible. Its primary function is to serve as an entry point for executing tasks. This file is responsible for importing and organizing all the tasks associated with a specific role. Instead of defining all tasks in a single file, they are structured explicitly across multiple files for better readability and modularity. The main.yml file ensures that these tasks are properly included and executed in a systematic manner when the role is applied.

---

### Explore the playbook.yml file


The playbook.yml file acts as the central component that organizes and manages different tasks. It encompasses all the roles that have been created and ensures their proper execution. This file plays a crucial role in streamlining the installation process of Kubernetes by implementing the predefined roles systematically. By structuring and coordinating these roles effectively, the playbook.yml file helps automate the deployment and setup of Kubernetes, making the process more efficient and manageable.

----

## Command to execute the playbook.yml file

Run the command in terminal
- sudo ansible-playbook playbook.yml

Then the command will get executed 

---

### Know about the inventory file and execution

The inventory will consists of different groups,servers and hosts name, if you want to avail, use the group name in the playbook.yml host name section and in order to execute the program.
Open the terminal and run the command below

- sudo ansible-playbook -i inventory.ini playbook.yml


--The above command will execute the playbook.yml which will lead to further exection of the all the roles and defined commands. The playbook.yml file after the execution will show different log files and processes which will let you know about the status of execution and debug the error if encountered
---- 
