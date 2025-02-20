# oai-cn5g-simulators-network-slicing

**OAI CN5G Network Slicing with Simulators in an Dockerised Environment**

![image](https://github.com/user-attachments/assets/7742dae4-7eba-4231-b114-00079f74ce42)

## Hardware Requirements

- **OS:** Ubuntu 22.04 LTS  
- **CPU:** 8 cores x86_64 @ 3.5 GHz  
- **RAM:** 32 GB  
- **Interfaces:** 2  

---

## Deployment Guides

For detailed steps, refer to the official documentation:

- [SA 5G Slicing Deployment](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/develop/docs/DEPLOY_SA5G_SLICING.md?ref_type=heads)  
- [Basic SA 5G Deployment](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/develop/docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md)  
- [Retrieve Official Images](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/develop/docs/RETRIEVE_OFFICIAL_IMAGES.md)  
- [NR SA Tutorial (OAI CN5G)](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md?ref_type=heads)  

Or follow the steps below for a quick setup.

---

## OAI CN5G Deployment

### Install Pre-requisites

```bash
sudo apt install -y git net-tools putty
```

### Install Docker

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to Docker group and reboot
sudo usermod -aG docker $(whoami)
reboot
```

### Clone OAI CN5G Repository

```bash
git clone --branch v2.1.0 https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git
cd oai-cn5g-fed
git checkout -f v2.1.0

# Sync components
./scripts/syncComponents.sh
```

### System Configuration

```bash
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
```

---

## Deploying OAI CN5G Slices

### Basic Network Slicing Setup

```bash
cd oai-cn5g-fed/docker-compose/
docker compose -f docker-compose-slicing-basic-nrf.yaml up -d
```

### Pull RAN simulators Images

```bash
docker pull rohankharade/gnbsim:latest
docker pull rohankharade/ueransim:latest
docker pull oaisoftwarealliance/oai-gnb:develop
docker pull oaisoftwarealliance/oai-nr-ue:develop

# Tag images
docker image tag rohankharade/gnbsim:latest gnbsim:latest
docker image tag rohankharade/ueransim:latest ueransim:latest
```

### Launch RAN Simulators

```bash
# Launch UERANSIM
docker compose -f docker-compose-slicing-ransim.yaml up -d ueransim

# Launch OAI gNB and NR-UE
docker compose -f docker-compose-slicing-ransim.yaml up -d oai-gnb oai-nr-ue1

# Launch GNBSIM
docker compose -f docker-compose-slicing-ransim.yaml up -d gnbsim
```

### Verify Running Containers

```bash
docker compose -f docker-compose-slicing-ransim.yaml ps -a
```

If oai-nr-ue exited after starting its container, remove `--sa` and `--nokrnmod` flags under `USE_ADDITIONAL_OPTIONS` parameter in `docker-compose-slicing-ransim.yaml` and re-run it.

### Monitor Logs

```bash
docker logs oai-amf -f
```

![image](https://github.com/user-attachments/assets/98d17cdd-8ca5-44f6-b09b-6d93c0d3b147)
