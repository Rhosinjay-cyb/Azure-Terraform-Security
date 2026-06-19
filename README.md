## Project Title

Shift-left Security Approach to the Deployment of Azure infrastructure with Third-party Tools

## Objective

The aim of this project is to demonstrate a shift-left security approach in the deployment of Azure infrastructure by implementing the security scanning of Iac with Checkov to identify and remediate misconfigurations before deployment. It also demonstrate the leveraging of tools in the Microsoft security stack to secure access to some of the deployed infrastructure.

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

Furthermore, Azure Bastion will be deployed for secured logon to the VM. Initially, the private key will be imported as a file from the local computer. Afterwards, Azure Key vault will be used to secure the key as an object and the private key will be integrated by the key vault.  Lastly, Checkov is initialized to scan the IaC, getting rid of misconfigurations in future deployments.

## Steps Taken

The project was accomplished in the following order

### Creation of a new directory for Terraform and configuration of its files via cloudshell

Azure Cloudshell was launched from Azure Portal, and a storage was specified to store created files. Although the integration of a storage account is optional, it keeps the file intact when the cloudshell session is restarted. In the cloudshell session a new directory was created and Terraform was initialized in the directory with 'Terraform init' command, followed with the configuration of .tf files.

![image](Images/A.Rule.png)

### Deployment of Azure infrastructure with Terraform

After the neccessary files are in place, the command 'Terraform plan' was used to generate a preview of changes Terraform intends to make. 


Afterwards, 'Terraform Apply' command is used to apply the changes generated from the last command basically creating the infrastructure.  

![image](Images/CR.Playbook.png)



Checking through the Azure portal to confirm if the infrastructure were successfully deployed. It was confirmed that all of the infrastructure specified in the .tf files were created ranging from the resource group to the virtual machine. 

### Configuration of security add-ons

Azure Bastion was deployed to enable secured access to the virtual machine.

Recall that the ssh public key has already been integrated in the VM by Azure. Here, the private key is specified to complete the authentication. 


Confirmed the successful logon to the Linux VM.


Rather than uploading the ssh private key during authentication which is quite risky. A key vault was created in Azure key vault to store the ssh private key as an object.



Subsequent logon to the Linux VM now utilize the object in the key vault, hence strengthening the security of the deployed VM.







### Implementation of security scanning and remediation

Checkov is the third-party tool used for the security scanning of the IaC. The process begin with the installation of Chekov. The installation could be confirmed by checking the version installed.



While in the Terraform-lab directory, the command 'Checkov -d .' was ran to scan the IaC. However, running the command outside the directory will require specifying the path to the directory. 









## Conclusion

This project succesfully demonstrates the implementation of playbooks in supporting incident response while enhancing the management of security operations. It also provides valuable insights into designing and orchestrating workflows in Azure Logic Apps for seamless integration with Microsoft Sentinel. Overall, the project highlights the end-to-end management of real-world security incidents covering the complete incident lifecycle from detection and containment to recovery and closure.

## Past Project

* Deployment of Azure Firewall for secure access and traffic control https://rhosinjay-cyb.github.io/Azure-Firewall/
* Deployment of Microsoft Sentinel to support cloud workload protection https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

