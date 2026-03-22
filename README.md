# Azure---VM---Deploy

Below is an **step-by-step Azure CLI guide** for **launching both Linux (Ubuntu Server) and Windows Server Azure VMs**, including **Azure VM Extensions (Custom Script)**.

This is **production-ready** and suitable for **labs, demos, CI/CD pipelines, and real projects**.

---

## 🔐 1️⃣ Login & Set Subscription

```bash
az login
az account set --subscription "<SUBSCRIPTION_ID>"
```

---

## 📦 2️⃣ Create Resource Group

```bash
az group create \
  --name myRG \
  --location eastus
```

---

# 🐧 Launch Azure VM – **Linux (Ubuntu Server)**

![Image](https://k21academy.com/wp-content/uploads/2020/07/Azure-portal.png)

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

![Image](https://res.cloudinary.com/canonical/image/fetch/f_auto%2Cq_auto%2Cfl_sanitize%2Cc_fill%2Cw_720/https%3A%2F%2Flh5.googleusercontent.com%2FM8fI_OhgDjiEawxsoUmMLDFw5QltILoGHjXczOI-H6nY9j3P_Oml9Ub8KkML7veDUaSTJmhU9T8Qf9uiTZxcJNesSGtdnZQdo5g_FN_roe9T_f_w1aPvcOPuIUmb5sS8x-MJic13)

## 3️⃣ Create Ubuntu Server VM

```bash
az vm create \
  --resource-group myRG \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username dhiraj \
  --admin-password '****' \
  --authentication-type password \
  --public-ip-sku Standard
  --os-disk-size-gb 30
```

### ✅ What this does

* Creates **Ubuntu Server 22.04 LTS**
* Generates SSH keys automatically
* Attaches a public IP
* Uses **Standard_B2s** VM size

---

## 4️⃣ Open SSH & HTTP Ports (Linux)

```bash
az vm open-port \
  --resource-group myRG \
  --name myVM \
  --port 22
```

```bash
az vm open-port \
  --resource-group myRG \
  --name myVM \
  --port 80
```

---

## 5️⃣ Linux VM – Custom Script Extension (Install NGINX)

```bash
az vm extension set \
  --resource-group myRG \
  --vm-name myVM \
  --name CustomScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{
    "commandToExecute": "sudo apt update && sudo apt install -y nginx"
  }'
```

---

# 🪟 Launch Azure VM – **Windows Server**

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

![Image](https://www.thomasmaurer.ch/wp-content/uploads/2021/11/Windows-Server-2022-Azure-Edition-scaled.jpg)

![Image](https://learn.microsoft.com/en-us/azure-stack/user/media/iaas-architecture-vm-windows/image1.png?view=azs-2506)

## 6️⃣ Create Windows Server VM

```bash
az vm create \
  --resource-group myRG \
  --name myVM \
  --image Win2022Datacenter \
  --size Standard_B2s \
  --admin-username azureadmin \
  --admin-password 'StrongPassword@123' \
  --public-ip-sku Standard \
  --os-disk-size-gb 64
```

⚠️ **Password requirements**

* Min 12 chars
* Uppercase + lowercase
* Number + special char

---

## 7️⃣ Open RDP & HTTP Ports (Windows)

```bash
az vm open-port \
  --resource-group myRG \
  --name myVM \
  --port 3389
```

```bash
az vm open-port \
  --resource-group rg-azure-vm-lab \
  --name vm-windows-01 \
  --port 80
```

---

## 8️⃣ Windows VM – Custom Script Extension (Install IIS)

```bash
az vm extension set \
  --resource-group rg-azure-vm-lab \
  --vm-name vm-windows-01 \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --settings '{
    "commandToExecute": "powershell Install-WindowsFeature -Name Web-Server"
  }'
```

---

## 📋 9️⃣ List VM Extensions

```bash
az vm extension list \
  --resource-group rg-azure-vm-lab \
  --vm-name vm-ubuntu-01 \
  --output table
```

```bash
az vm extension list \
  --resource-group rg-azure-vm-lab \
  --vm-name vm-windows-01 \
  --output table
```

---

## 🔍 🔟 Get Public IP Addresses

```bash
az vm list-ip-addresses \
  --resource-group rg-azure-vm-lab \
  --output table
```

---

## 🧹 1️⃣1️⃣ Clean-up Resources

```bash
az group delete \
  --name rg-azure-vm-lab \
  --yes --no-wait
```

---

## 🎯 Best Practices (Production)

* Use **cloud-init** for Linux bootstrapping
* Store scripts in **GitHub / Azure Storage**
* Use **Azure Key Vault** for secrets
* Use **Bicep / Terraform** instead of raw CLI
* Enable **Azure Monitor Agent**
* Restrict NSG ports (avoid `0.0.0.0/0`)

---
## 🔹 Azure VM Initialization Options – Big Picture

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

![Image](https://labresources.whizlabs.com/391a18a3eebf32fc6e566ec5de369a58/cse.png)

Azure provides **two primary mechanisms** to configure VMs at boot or post-deployment:

| Method            | Runs When       | Best For                                 |
| ----------------- | --------------- | ---------------------------------------- |
| **Cloud-Init**    | First boot only | OS-level initialization (Linux)          |
| **VM Extensions** | Anytime         | Post-deploy automation (Linux + Windows) |

---

# 1️⃣ Azure Cloud-Init (Linux Only)

### 🔹 What is Cloud-Init?

**Cloud-Init** is a **native Linux initialization system** that runs **once on first boot**.

✔ Runs **before SSH login**
✔ Faster than extensions
✔ Ideal for **immutable infrastructure**

---

## 🔹 Typical Cloud-Init Use Cases

* Create users & SSH keys
* Install packages
* Configure hostname
* Write config files
* Run bootstrap scripts

---

## 🔹 Cloud-Init YAML Example

```yaml
#cloud-config
package_update: true
package_upgrade: true

packages:
  - nginx
  - git
  - docker.io

users:
  - name: devops
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1...

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - echo "Cloud-init completed" > /var/log/cloud-init-done.log
```

---

## 🔹 Deploy VM with Cloud-Init (Azure CLI)

```bash
az vm create \
  --resource-group rg-demo \
  --name linux-vm-cloudinit \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.yaml
```

📌 **Note**:

* `--custom-data` = cloud-init YAML
* Runs **only once** (on first boot)

---

## 🔹 Verify Cloud-Init Execution

```bash
cloud-init status
cloud-init analyze
cat /var/log/cloud-init-output.log
```

---

# 2️⃣ Azure VM Script Extensions (Advanced)

### 🔹 What are VM Extensions?

VM Extensions are **agents installed on the VM** that allow you to:

✔ Run scripts anytime
✔ Re-run scripts
✔ Integrate with CI/CD
✔ Configure post-deployment

---

## 🔹 Common Azure Script Extensions

| Extension                   | OS              |
| --------------------------- | --------------- |
| **Custom Script Extension** | Linux / Windows |
| **DSC Extension**           | Windows         |
| **Chef / Puppet / Ansible** | Linux / Windows |
| **Azure Monitor Agent**     | Both            |

---

# 3️⃣ Custom Script Extension – Linux

![Image](https://ochzhen.com/assets/img/azure-custom-script-extension-linux/assigned-identity-to-vmss.png)

![Image](https://media.licdn.com/dms/image/v2/C5612AQEHKEJUPWuV2A/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1584082178681?e=2147483647\&t=SBo0aofLnNj49QPrPJH6em-bhmHZ30M_dr8rxPg5NyU\&v=beta)

![Image](https://learn.microsoft.com/en-us/azure/architecture/virtual-machines/media/baseline-network-egress.svg)

### 🔹 Use Cases

* Install applications
* Pull GitHub scripts
* Apply patches
* Reconfigure VM after creation

---

## 🔹 Linux Custom Script Extension (Inline)

```bash
az vm extension set \
  --resource-group rg-demo \
  --vm-name linux-vm \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{
    "commandToExecute": "apt update && apt install -y nginx"
  }'
```

---

## 🔹 Linux Script from GitHub / Storage

```bash
az vm extension set \
  --resource-group rg-demo \
  --vm-name linux-vm \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{
    "fileUris": ["https://raw.githubusercontent.com/org/repo/main/setup.sh"],
    "commandToExecute": "bash setup.sh"
  }'
```

---




## 📌 1️⃣ Azure CLI

```bash
az group create --name MyResourceGroup --location eastus

az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```
```
az vm create \
  --resource-group LAMP-ResourceGroup \
  --name LampVM \
  --image Ubuntu2404 \
  --admin-username atul \
  --generate-ssh-keys \
  --size Standard_B1s
```

---

## 📌 2️⃣ Azure PowerShell

```powershell
New-AzResourceGroup -Name "MyResourceGroup" -Location "EastUS"

New-AzVM `
  -ResourceGroupName "MyResourceGroup" `
  -Name "MyVM" `
  -Location "EastUS" `
  -VirtualNetworkName "MyVNet" `
  -SubnetName "MySubnet" `
  -SecurityGroupName "MyNSG" `
  -PublicIpAddressName "MyPublicIP" `
  -OpenPorts 80,3389
```

---

## 📌 3️⃣ ARM Template (JSON)

**azuredeploy.json**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": { "type": "string" },
    "adminUsername": { "type": "string" },
    "adminPassword": { "type": "securestring" }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2022-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": { "vmSize": "Standard_DS1_v2" },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          },
          "osDisk": { "createOption": "FromImage" }
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(parameters('vmName'),'-nic'))]"
            }
          ]
        }
      }
    }
  ]
}
```

Deploy it:

```bash
az deployment group create --resource-group MyResourceGroup --template-file azuredeploy.json --parameters vmName=MyVM adminUsername=azureuser adminPassword=YourPassword123!
```

---
