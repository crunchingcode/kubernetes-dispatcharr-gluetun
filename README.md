# kubernetes-dispatcharr-gluetun

Ansible playbook and variable files for deploying Dispatcharr with Gluetun VPN on Kubernetes.

## Overview

This repository contains Ansible playbooks to deploy:
- **Gluetun**: A VPN client container that routes traffic through various VPN providers
- **Dispatcharr**: A service that routes its traffic through the Gluetun VPN

## Prerequisites

Before running the playbook, ensure you have:

1. **Ansible** installed (version 2.9 or higher)
   ```bash
   pip install ansible
   ```

2. **Kubernetes cluster** accessible with `kubectl` configured

3. **Python Kubernetes client** installed
   ```bash
   pip install kubernetes
   ```

4. **Ansible Kubernetes collection** installed
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

## Installation

### From a Jump Box (Ansible Host)

I install all my Kubernetes pods/containers via Ansible from a jump box. Others have different setups but those should be able to figure out what to change.

To install, you run:

```bash
# 1. Clone this repository
git clone https://github.com/crunchingcode/kubernetes-dispatcharr-gluetun.git
cd kubernetes-dispatcharr-gluetun

# 2. Install Ansible collection dependencies
ansible-galaxy collection install -r requirements.yml

# 3. Configure your VPN settings in vars/main.yml
# Edit the file and add your VPN credentials and preferences
nano vars/main.yml

# 4. Run the playbook
ansible-playbook -i inventory.yml deploy.yml
```

### Quick Start

For a quick deployment with default settings:

```bash
ansible-playbook -i inventory.yml deploy.yml
```

## Configuration

All configuration is managed through `vars/main.yml`. Key settings include:

### Namespace
```yaml
namespace: dispatcharr-gluetun
```

### VPN Configuration (Gluetun)
```yaml
gluetun:
  vpn_service_provider: nordvpn  # or expressvpn, surfshark, etc.
  vpn_type: openvpn              # or wireguard
  openvpn_user: "your_username"
  openvpn_password: "your_password"
  server_countries: "United States"
```

### Dispatcharr Configuration
```yaml
dispatcharr:
  image: dispatcharr/dispatcharr:latest
  port: 8080
  env:
    LOG_LEVEL: "info"
```

### Service Type
```yaml
service:
  type: ClusterIP  # Options: ClusterIP, NodePort, LoadBalancer
```

## Directory Structure

```
.
├── deploy.yml                    # Main Ansible playbook
├── inventory.yml                 # Ansible inventory file
├── requirements.yml              # Ansible collection dependencies
├── vars/
│   └── main.yml                 # Configuration variables
└── templates/
    ├── gluetun-deployment.yml.j2       # Gluetun Kubernetes deployment
    ├── gluetun-service.yml.j2          # Gluetun Kubernetes service
    ├── dispatcharr-deployment.yml.j2   # Dispatcharr Kubernetes deployment
    └── dispatcharr-service.yml.j2      # Dispatcharr Kubernetes service
```

## Usage

### Deploy
```bash
ansible-playbook -i inventory.yml deploy.yml
```

### Verify Deployment
```bash
kubectl get pods -n dispatcharr-gluetun
kubectl get services -n dispatcharr-gluetun
```

### Check Logs
```bash
# Gluetun logs
kubectl logs -n dispatcharr-gluetun deployment/gluetun

# Dispatcharr logs
kubectl logs -n dispatcharr-gluetun deployment/dispatcharr
```

### Update Configuration
1. Edit `vars/main.yml` with your new settings
2. Re-run the playbook:
   ```bash
   ansible-playbook -i inventory.yml deploy.yml
   ```

## Customization for Different Setups

While I use a jump box with Ansible, you can adapt this for other setups:

### Using kubectl directly
Convert the Jinja2 templates to plain YAML and apply with:
```bash
kubectl apply -f templates/
```

### Using Helm
Convert the templates to a Helm chart structure

### Using GitOps (ArgoCD/Flux)
Commit the rendered manifests to a Git repository

## Supported VPN Providers

Gluetun supports many VPN providers including:
- NordVPN
- ExpressVPN
- Surfshark
- Private Internet Access
- Mullvad
- ProtonVPN
- And many more...

See the [Gluetun documentation](https://github.com/qdm12/gluetun) for a complete list.

## Troubleshooting

### VPN Connection Issues
```bash
# Check Gluetun logs for connection status
kubectl logs -n dispatcharr-gluetun deployment/gluetun
```

### Pod Not Starting
```bash
# Describe the pod to see events
kubectl describe pod -n dispatcharr-gluetun -l app=gluetun
kubectl describe pod -n dispatcharr-gluetun -l app=dispatcharr
```

### Network Issues
Ensure the dispatcharr container is correctly configured to use the Gluetun proxy:
- HTTP_PROXY environment variable should point to gluetun service
- Check network policies if applicable

## License

This project is provided as-is for community use.

## Contributing

Feel free to submit issues and pull requests for improvements.
