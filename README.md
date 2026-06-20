## Project Title

Implementing Shift-left Security for Azure Infrastructure Deployment using Terraform and Checkov

## Objective

The aim of this project is to demonstrate a shift-left security approach to the deployment of Azure infrastructure by integrating security scanning into the infrastructure as code (Iac) development lifecycle. Using Checkov, the project performs static analysis of Terraform configutations to identify and remediate security misconfigurations before deployment, thereby reducing risk and improving infrastructure security. Additionally, the project showcases the use of security capabilities within the microsoft security stack to implement secure accesscontrols for deployed Azure resources, promoting adherence to cloud security best practices and enhancing the overallsecurity posture of the environment.

## Tools Used

Terraform, Checkov, Azure Cloudshell, Azure Key Vault, Azure Bastion


## Lab Setup
* Creation of a new directory for Terraform and configuration of its files via cloudshell
* Deployment of Azure infrastructure with Terraform
* Configuration of security add-ons
* Implementation of security scanning and remediation

## Background Information
The project is aimed at enforcing a shift-left security principle to support both infrastructure and DevOps engineering. These aspects of engineering play a vital role in software development, and infrastructure as code (IaC) is common to both of them. Due to long line of code for deployment, manual scanning makes the code succesptible for vulnerabilities. However, applying an automated security scanning allows the misconfigurations tobe detected and remediated before deployment.

Consequently, this project aim at deploying Azure Infrastructure (VMs, Vnets, NSGs, RG, Subnets) with Terraform and implementing security scanning of IaC to get rid of misconfigurations. The main infrastructure in this project is the Linux VM. In this project we will be using a ssh key pair for authenticating against the Linux VM. While the ssh key pair is generated, the public key will sent to Azure by Terraform during VM creation while the private key will kept for logon.

![image](Images/keygen.png)

Furthermore, Azure Bastion will be deployed for secured logon to the VM. Initially, the private key will be imported as a file from the local computer. Afterwards, Azure Key vault will be used to secure the key as an object and the private key will be integrated by the key vault.  Lastly, Checkov is initialized to scan the IaC, getting rid of misconfigurations in future deployments.

## Steps Taken

The project was accomplished in the following order

### Creation of a new directory for Terraform and configuration of its files via cloudshell

Azure Cloudshell was launched from Azure Portal, and a storage was specified to store created files. Although the integration of a storage account is optional, it keeps the file intact when the cloudshell session is restarted. In the cloudshell session a new directory was created and Terraform was initialized in the directory with 'Terraform init' command, followed with the configuration of .tf files.

![image](Images/edit_folder.png)

![image](Images/Init.png)

### Deployment of Azure infrastructure with Terraform

After the neccessary files are in place, the command 'Terraform plan' was used to generate a preview of changes Terraform intends to make. 

![image](Images/plan1.png)
![image](Images/plan2.png)

Afterwards, 'Terraform Apply' command is used to apply the changes generated from the last command basically creating the infrastructure.  
![image](Images/plan3.png)
![image](Images/plan4.png)

Checking through the Azure portal to confirm if the infrastructure were successfully deployed. It was confirmed that all of the infrastructure specified in the .tf files were created ranging from the resource group to the virtual machine. 

![image](Images/result.png)
![image](Images/result2.png)

### Configuration of security add-ons

Azure Bastion was deployed to enable secured access to the virtual machine.

![image](Images/create_bastion.png)

Recall that the ssh public key has already been integrated in the VM by Azure. Here, the private key is specified to complete the authentication. 

![image](Images/access_vm.png)

Confirmed the successful logon to the Linux VM.

![image](Images/access_vm2.png)

Rather than uploading the ssh private key during authentication which is quite risky. A key vault was created in Azure key vault to store the ssh private key as a secret.

![image](Images/createkeyvault.png)

To create a secret in the key vault, the user was assigned the appropriate RBAC role.

![image](Images/rbac.png)
![image](Images/rbac2.png)

Afterwards, the secret is now created, via Azure Cloudshell

![image](Images/create_secret2.png)

The creation of the secret was confirmed from Azure portal.

![image](Images/succ_key.png)

Subsequent logon to the Linux VM now utilize the object in the key vault, hence strengthening the security of the deployed VM.

![image](Images/conn.png)

Confirmed the successful logon to the Linux VM.

![image](Images/succ_conn.png)

### Implementation of security scanning and remediation

Checkov is the third-party tool used for the security scanning of the IaC. The process begin with the installation of Chekov. The installation was confirmed by checking the version installed.

![image](Images/checkov.png)
![image](Images/checkov2.png)

While in the Terraform-lab directory, the command 'Checkov -d .' was ran to scan the IaC. However, running the command outside the directory will require specifying the path to the directory. From the scan result, it shows we have three failed checks to remediate.

![image](Images/sec_scan.png)

The scan result is mainly grouped into two categories, passed checks and failed checks. 
The passed checks indicate that the configurations are accurate,

![image](Images/passed.png)

while the failed checks indicate a misconfiguration in the IaC.

![image](Images/failed.png)
![image](Images/failed2.png)

Remediating the first miconfiguration, 'ensure that SSH access is restricted from the internet,' the source address prefix is restricted to the AzureBastionSubnet. This ensures that only connections established from the subnet could connect to the VM.

![image](Images/restrict.png)

The next misconfiguration 'ensure VNET subnet is configured with a network security group (NSG)' was remediated by assosciating the subnet to a NSG. This allows the assocuaition of relevant NSG security rule to the subnet, thereby enhancing the security of the subnet.

![image](Images/assoc.png)

The last misconfiguration 'ensure virtual machine extensions are not installed' was investigted. It was observed that the VM has no extension, hence the error was skipped.

Having remediated all the misconfigurations, the IaC was re-scanned, and no error was detected.

![image](Images/remediated.png)


## Conclusion
This project succesfully demonstrate a shift-left security approach to provisioning infrastructure. In a wider context, this relates to securing a software development lifecycle (SDLC). Additionally, the skills exhibited on this project represents a part of the  skills required of a DevSecOps or a security engineer collaborating with the DevOps team to work effectively.

## Past Project

* Deployment of Azure Firewall for secure access and traffic control https://rhosinjay-cyb.github.io/Azure-Firewall/
* Deployment of Microsoft Sentinel to support cloud workload protection https://rhosinjay-cyb.github.io/Microsoft-Sentinel/
* Automating Security Incident Response with Playbooks https://rhosinjay-cyb.github.io/Incident-Response-with-Playbooks/
