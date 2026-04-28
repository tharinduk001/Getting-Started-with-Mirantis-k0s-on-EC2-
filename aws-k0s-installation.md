## Deploying a Single-Node k0s Cluster on AWS EC2

This guide outlines the end-to-end process for provisioning an AWS EC2 instance, deploying a lightweight k0s Kubernetes cluster, establishing remote access via Lens IDE, and running a test deployment.

#### Phase 1: Infrastructure Provisioning (AWS EC2)

Before installing k0s, spin up an EC2 instance with the following minimum specifications to ensure the control plane and workloads run smoothly.

- Operating System: Ubuntu Server (22.04 LTS or 24.04 LTS)
- Instance Type: t3.small (or any instance with at least 2GB of RAM)
- Storage (EBS): 20 GB General Purpose SSD (gp2/gp3)
- Network Security Group (Inbound Rules):
- SSH (Port 22): Source: My IP (for secure terminal access)
- Custom TCP (Port 6443): Source: My IP (Required for Kubernetes API / Lens IDE access)

#### Phase 2: k0s Installation

Connect to your newly provisioned EC2 instance via SSH and execute the following commands to install and configure k0s as a single-node cluster (acting as both controller and worker).

1.  Download the k0s binary:
    `curl -sSLf [https://get.k0s.sh](https://get.k0s.sh) | sudo sh`
2.  Install k0s as a system service:
    `sudo k0s install controller --single`
3.  Start the cluster:
    `sudo k0s start`
4.  Verify the service is running:
    `sudo k0s status`

#### Phase 3: Cluster Verification

Give the system about 60 seconds to initialize the core services. Then, verify the health of your node and system pods using the built-in k0s `kubectl` wrapper.

1.  Check node readiness:
    `sudo k0s kubectl get nodes`

2.  Check the status of core system pods:
    `sudo k0s kubectl get pods -A`

(Ensure all pods are in a Running state).

#### Phase 4: Remote Access configuration (Lens IDE / Windows)

To manage the cluster from your local Windows machine using Lens IDE or a local kubectl installation, you need to extract and modify the cluster's configuration file.

1. Extract the Kubeconfig from the server:
   Run this command on your EC2 instance and copy the entire output (starting from `apiVersion: v1` to the end):

`sudo cat /var/lib/k0s/pki/admin.conf`

2. Configure your local Windows environment:

- Open a text editor (like Notepad or VS Code).
- Paste the copied configuration text.
- Locate the `server`: line and update `localhost` to your EC2 instance's Public IPv4 address:
  - Change: server: `https://localhost:6443`
  - To: server: `https://<YOUR_EC2_PUBLIC_IP>:6443`
- TLS Bypass (For Sandbox Environments): To prevent x509 certificate errors when connecting via the public IP, locate the `certificate-authority-data` line, delete it, and replace it with `insecure-skip-tls-verify: true`.
- Save the file to your `.kube` directory as `config`:
  - Path: `C:\Users\YourUsername\.kube\config`

3. Connect: Open Lens IDE. It will automatically detect the configuration in your `.kube` folder and connect to your cluster.

#### Phase 5: Test Deployment

Verify that the cluster can successfully schedule and expose workloads by deploying a simple NGINX web server.

Run the following commands from your local terminal (or the Lens IDE terminal):

1. Create the NGINX deployment:
   `kubectl create deployment nginx-test --image=nginx`

2. Expose the deployment:
   `kubectl expose deployment nginx-test --port=80 --type=NodePort`

3. Retrieve the exposed port:
   `kubectl get services`

(Look for the nginx-test service and note the assigned 5-digit NodePort. You can access the NGINX splash page by navigating to `http://<YOUR_EC2_PUBLIC_IP>:<NODE_PORT>` in your browser, provided that port is opened in your AWS Security Group).
