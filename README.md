# Network Definitions Helm Chart

A declarative Helm chart for deploying NetworkAttachmentDefinitions for Multus CNI, optimized for VoIP and telecommunications infrastructure.

## 📋 Overview

This Helm chart deploys NetworkAttachmentDefinitions (NADs) for use with an existing Multus CNI installation. It provides pre-configured network definitions for common VoIP services including FreeSWITCH, Kamailio, RTPEngine, and associated databases.

**Note:** This chart **only** deploys NetworkAttachmentDefinitions. Multus CNI must already be installed in your cluster.

## ✨ Features

- 🎯 Declarative NetworkAttachmentDefinition management
- 🔧 Pre-configured for common VoIP services
- 📦 Easy customization via values.yaml
- 🌐 MACVLAN networking with IPAM
- 🔄 Support for multiple namespaces
- ⚡ Enable/disable networks individually
- 🎨 Custom DNS and gateway configuration per network

## 📋 Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- **Multus CNI already installed and running**
- NetworkAttachmentDefinition CRD installed
- Network interface available on nodes (default: ens192)

### Verify Prerequisites

```bash
# Check Multus is running
kubectl get daemonset -n kube-system | grep multus

# Verify NetworkAttachmentDefinition CRD exists
kubectl get crd networkattachmentdefinitions.k8s.cni.cncf.io

# Check available network interfaces on nodes
kubectl get nodes -o wide
```

## 📊 Included Network Definitions

| Service | Network Name | IP Range | Namespace | Description |
|---------|-------------|----------|-----------|-------------|
| FreeSWITCH | macvlan-freeswitch-45-55 | 192.168.9.45-55 | default | VoIP PBX system |
| Kamailio | macvlan-kamailio-38-40 | 192.168.9.38-40 | default | SIP proxy server |
| RTPEngine | macvlan-rtpengine-41-43 | 192.168.9.41-43 | default | RTP media proxy |
| MySQL | macvlan-mysql-44-46 | 192.168.9.44-46 | default | Database server |
| Redis | macvlan-redis-47-49 | 192.168.9.47-49 | default | Cache server |
| HPA | macvlan-hpa-50-52 | 192.168.9.50-52 | default | Homer/Prometheus/Alertmanager |
| MongoDB | macvlan-mongodb-53-55 | 192.168.9.53-55 | default | NoSQL database |
| Homer | macvlan-homer-56-58 | 192.168.9.56-58 | default | SIP capture server |

## 🚀 Installation

### Quick Start

```bash
# Clone the repository
git clone https://github.com/allwinrajan/network-definitions-helm.git
cd network-definitions-helm

# Install with default values
helm install network-definitions . -n default
```

### Custom Installation

```bash
# Create custom values file
cat > my-values.yaml <<EOF
global:
  masterInterface: eth0  # Change to match your interface
  subnet: 10.0.0.0/24
  gateway: 10.0.0.1
  dns:
    nameservers:
      - 8.8.8.8
      - 10.0.0.1

definitions:
  freeswitch:
    enabled: true
    rangeStart: 10.0.0.45
    rangeEnd: 10.0.0.55
  
  kamailio:
    enabled: true
    rangeStart: 10.0.0.38
    rangeEnd: 10.0.0.40
  
  # Disable networks you don't need
  mongodb:
    enabled: false
EOF

# Install with custom values
helm install network-definitions . -n default -f my-values.yaml
```

### Install Specific Networks Only

```bash
# Only deploy FreeSWITCH and Kamailio networks
cat > minimal-values.yaml <<EOF
definitions:
  freeswitch:
    enabled: true
  kamailio:
    enabled: true
  rtpengine:
    enabled: false
  mysql:
    enabled: false
  redis:
    enabled: false
  hpa:
    enabled: false
  mongodb:
    enabled: false
  homer:
    enabled: false
EOF

helm install network-definitions . -f minimal-values.yaml
```

## ⚙️ Configuration

### Global Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.masterInterface` | Host network interface name | `ens192` |
| `global.subnet` | Network subnet CIDR | `192.168.9.0/24` |
| `global.gateway` | Default gateway IP | `192.168.9.1` |
| `global.cniVersion` | CNI specification version | `0.3.1` |
| `global.type` | CNI plugin type | `macvlan` |
| `global.mode` | MACVLAN mode | `bridge` |
| `global.dns.nameservers` | DNS servers list | `[8.8.8.8, 192.168.9.1]` |
| `global.dns.domain` | DNS domain | `local` |
| `global.dns.search` | DNS search domains | `[local]` |

### Network Definition Settings

Each network definition supports:

| Parameter | Description | Required |
|-----------|-------------|----------|
| `enabled` | Enable/disable this network | Yes |
| `namespace` | Kubernetes namespace | Yes |
| `rangeStart` | Start IP address | Yes |
| `rangeEnd` | End IP address | Yes |
| `description` | Human-readable description | Yes |
| `masterInterface` | Override global interface | No |
| `gateway` | Override global gateway | No |
| `subnet` | Override global subnet | No |
| `dns` | Override global DNS settings | No |

### Example: Adding Custom Networks

```yaml
definitions:
  # Add your custom network
  custom-app:
    enabled: true
    namespace: production
    rangeStart: 192.168.9.100
    rangeEnd: 192.168.9.110
    description: Custom Application Network
    # Optional overrides
    masterInterface: eth1
    gateway: 192.168.9.254
    dns:
      nameservers:
        - 1.1.1.1
        - 8.8.8.8
```

## 📝 Usage Examples

### Basic Pod with Network

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: freeswitch
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-freeswitch-45-55
spec:
  containers:
  - name: freeswitch
    image: freeswitch:latest
    ports:
    - containerPort: 5060
```

### Pod with Multiple Networks

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: voip-gateway
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-kamailio-38-40,macvlan-rtpengine-41-43
spec:
  containers:
  - name: gateway
    image: voip-gateway:latest
```

### Pod with Specific IP Request

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: freeswitch-primary
  annotations:
    k8s.v1.cni.cncf.io/networks: |
      [
        {
          "name": "macvlan-freeswitch-45-55",
          "ips": ["192.168.9.45"]
        }
      ]
spec:
  containers:
  - name: freeswitch
    image: freeswitch:latest
```

### Deployment with Network

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kamailio
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kamailio
  template:
    metadata:
      labels:
        app: kamailio
      annotations:
        k8s.v1.cni.cncf.io/networks: macvlan-kamailio-38-40
    spec:
      containers:
      - name: kamailio
        image: kamailio:latest
        ports:
        - containerPort: 5060
```

### StatefulSet with Network

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
      annotations:
        k8s.v1.cni.cncf.io/networks: macvlan-mysql-44-46
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### Multiple Networks with Interface Names

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-network-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: |
      [
        {
          "name": "macvlan-kamailio-38-40",
          "interface": "net1",
          "ips": ["192.168.9.38"]
        },
        {
          "name": "macvlan-rtpengine-41-43",
          "interface": "net2",
          "ips": ["192.168.9.41"]
        }
      ]
spec:
  containers:
  - name: app
    image: my-app:latest
```

## 🔍 Verification

### List All NetworkAttachmentDefinitions

```bash
# All namespaces
kubectl get network-attachment-definitions --all-namespaces

# Short form
kubectl get net-attach-def -A

# Specific namespace
kubectl get net-attach-def -n default
```

### Describe a Specific Network

```bash
kubectl describe net-attach-def macvlan-freeswitch-45-55 -n default
```

### View Network Configuration

```bash
kubectl get net-attach-def macvlan-freeswitch-45-55 -n default -o yaml
```

### Check Pod Network Status

```bash
# View pod annotations
kubectl get pod <pod-name> -o jsonpath='{.metadata.annotations}' | jq

# View network status
kubectl get pod <pod-name> -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/network-status}' | jq
```

### Verify Pod IPs

```bash
# Get all IPs for a pod
kubectl exec <pod-name> -- ip addr show

# Check specific interface
kubectl exec <pod-name> -- ip addr show net1
```

## 🔄 Upgrading

### Modify Existing Networks

```bash
# Edit values.yaml or create new values file
helm upgrade network-definitions . -n default -f updated-values.yaml
```

### Add New Networks

```bash
# Add new definition to values.yaml
definitions:
  new-service:
    enabled: true
    namespace: default
    rangeStart: 192.168.9.200
    rangeEnd: 192.168.9.210
    description: New Service Network

# Upgrade
helm upgrade network-definitions . -n default
```

### Disable Networks

```bash
# Set enabled: false for networks you want to remove
definitions:
  mongodb:
    enabled: false

# Upgrade (this will delete the NetworkAttachmentDefinition)
helm upgrade network-definitions . -n default
```

## 🗑️ Uninstallation

```bash
# Uninstall the release
helm uninstall network-definitions -n default

# Verify removal
kubectl get net-attach-def -n default
```

**Warning:** Uninstalling will remove all NetworkAttachmentDefinitions. Ensure no pods are using them before uninstalling.

## 🐛 Troubleshooting

### Issue: CRD Not Found

```bash
Error: networkattachmentdefinitions.k8s.cni.cncf.io not found
```

**Solution:** Install Multus CNI first or manually install the CRD:

```bash
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset.yml
```

### Issue: Pod Not Getting Secondary IP

**Check Network Attachment:**

```bash
kubectl describe pod <pod-name> | grep "k8s.v1.cni.cncf.io/networks"
```

**Check Multus Logs:**

```bash
kubectl logs -n kube-system -l app=multus
```

**Verify Network Definition Exists:**

```bash
kubectl get net-attach-def <network-name> -n <namespace>
```

### Issue: IP Address Conflicts

**Solution:** Review your IP ranges and ensure they don't overlap:

```bash
# List all network definitions and their ranges
kubectl get net-attach-def -A -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,CONFIG:.spec.config | grep -E "rangeStart|rangeEnd"
```

### Issue: Wrong Interface

**Check Node Interfaces:**

```bash
# SSH to node and check interfaces
ip link show

# Update values.yaml with correct interface
global:
  masterInterface: eth0  # or ens192, ens33, etc.
```

### Issue: DNS Not Working

**Test DNS from Pod:**

```bash
kubectl exec -it <pod-name> -- nslookup google.com

# Check nameserver configuration
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
```

**Update DNS Settings:**

```yaml
global:
  dns:
    nameservers:
      - 8.8.8.8
      - 1.1.1.1
```

### Debug Mode

Enable verbose logging in Multus (if you have access to Multus config):

```yaml
{
  "logLevel": "verbose",
  "logFile": "/var/log/multus.log"
}
```

View logs:

```bash
kubectl logs -n kube-system <multus-pod-name> --tail=100
```

## 📖 Advanced Configuration

### Per-Network DNS Override

```yaml
definitions:
  custom-network:
    enabled: true
    namespace: default
    rangeStart: 192.168.9.100
    rangeEnd: 192.168.9.110
    description: Custom Network with Different DNS
    dns:
      nameservers:
        - 1.1.1.1
        - 1.0.0.1
      domain: custom.local
      search:
        - custom.local
        - cluster.local
```

### Multiple Namespaces

```yaml
definitions:
  prod-app:
    enabled: true
    namespace: production
    rangeStart: 192.168.9.50
    rangeEnd: 192.168.9.60
    description: Production Network
  
  dev-app:
    enabled: true
    namespace: development
    rangeStart: 192.168.10.50
    rangeEnd: 192.168.10.60
    description: Development Network
    subnet: 192.168.10.0/24
    gateway: 192.168.10.1
```

### Common Labels and Annotations

```yaml
advancedConfig:
  commonLabels:
    environment: production
    team: voip
    managed-by: platform-team
  
  commonAnnotations:
    company: example-corp
    cost-center: "12345"
```

## 📚 Additional Resources

- [Multus CNI Documentation](https://github.com/k8snetworkplumbingwg/multus-cni/blob/master/docs/how-to-use.md)
- [NetworkAttachmentDefinition Spec](https://github.com/k8snetworkplumbingwg/network-attachment-definition-client)
- [MACVLAN CNI Plugin](https://www.cni.dev/plugins/current/main/macvlan/)
- [Kubernetes Network Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

## 👤 Maintainer

- **allwinrajan** - [GitHub](https://github.com/allwinrajan)

## ⭐ Support

If you find this chart helpful, please give it a star on GitHub!

---

**Made with ❤️ for the VoIP and Kubernetes community**