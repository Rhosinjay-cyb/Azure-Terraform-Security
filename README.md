## Project Title

Shift-left Security Approach to the Deployment of Azure infrastructure with Third-party Tools

## Objective

The aim of this project is to demonstrate a shift-left security approach in the deployment of Azure infrastructure by implementing the security scanning of Iac with Checkov to identify and remediate misconfigurations before deployment. It also demonstrate the leveraging of tools in the Microsoft security stack to secure access to some of the deployed infrastructure.

## Tools Used

Terraform, Checkov, Cloudshell, Azure Key Vault, Azure Bastion


## Lab Setup
* Creation of a new directory for Terraform and configuration of its files via cloudshell
* Deployment of Azure infrastructure with Terraform
* Configuration of security add-ons
* Implementation of security scanning and remediation


## Steps Taken

The project was accomplished in the following order

### Creation of a new directory for Terraform and configuration of its files via cloudshell

Cloudshell was launched from Azure Portal, and a storage was specified to store created files. Although the creating a storage 

![image](Images/A.Rule.png)

### Creation of the Azure Logic App playbook

The playbook is created with Microsoft Sentinel incident as the trigger. While creating the playbook the subscription and resource group is specified alongside the tier of the Azure Logic App.

![image](Images/CR.Playbook.png)

### Designing the workflow of the playbook

Following the creation of the playbook, the the workflow of the playbook from trigger to the last action was designed with the logic app designer blade.

![image](Images/Workflow.png)

The trigger for the workflow is Microsoft Sentinel Incident because the playbook is designed to run when an incident is created in Microsoft Sentinel.

To collect the entities in the incident, these variables of array data type are initialized to collect and store the entities. 

Note: Entity mapping is the responsibility of the security team while creating the analytics rule, mapping the right entity to the analytics rule assists in enriching incident investigation.

![image](Images/INI.Variables.png)

The for each loop has two seperate conditions which were used to collect the entities. The first conditons appends IPs to the array IPList if the entity kind equals Ip while the under condition appends Username to the array UserList if the entity kind equals Account.

![image](Images/For.Each.png)

The above actions ONLY implements the collection of entities from the security incident.

The next action was used to create a network securtiy group (NSG) security rule to block all inbound connection to the Sales-VM including incoming RDP traffic. The Azure Resource Manager connector is used for this action and 'Create or update resource' was choosen among the available options. The parameters were filled accordingly, resource explorer would be quite helpful in getting the short resource id. Location and properties were selected among the advanced parameters and equally specified as well. The values in the properties field will now be used to create a NSG security rule by updating the Sales-VM-nsg which will block inbound connections to the Sales-VM. Blocking the inbound traffic will prevent the attacker from reconnecting.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a network contributor role at the NSG of the virtual machine (Sales-VM-nsg) enforcing the principle of least privilege.

![image](Images/Blk.NSG.png)

The next action in the workflow is the closing of the attacker's remote. This action also ensured that only the malicious session was closed while other sessions were skipped preventing the disruption of production operation. This action was implemented with the HTTP connector, which sent a web request to the API of Azure Resource Manager (ARM), the ARM validated the request and allowed the runCommand service to utilize the Azure VM agent to exexute the powershell script specified in the body of the HTTP request, basically to close the attacker's remote session.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a Virtual Machine contributor role at the scope of the resource, Sales-VM.

![image](Images/Close.MRDP.png)

The last of the actions on the workflow was to send an email notification to relevant members of the security operations team for incident review and other neccessary actions. Aside alerting the team, the email also provide a summary of the attack at a glance with details that includes the entities extracted from the security incident.

![image](Images/Send.Email.png)

### Integration of playbook with Microsoft Sentinel Incident

An automation rule was created to integrate the playbook with the Microsoft Sentinel Incident. The generation of a security incident starts with a security alert, and the analytics rule is reponsible for firing the security alert. Hence, the earlier created analytics rule was edited, and an automation rule was created at the 'Automated Response' tab of the analytics rule. This configuration now allows the playbook to be triggered once the incident is generated.
There were two actions in the automation rule, the first one was to assign the security incident to a member of the SecOps team while the other action was to run the playbook.

![image](Images/Aut.Rule1.png)
![image](Images/Aut.Rule2.png)

A major prequisite that was implemented for seamless interaction between Microsoft Sentinel and the playbook was to configure the playbook permission of Microsoft Sentinel at resource group level. This configuration automatically assigns the Microsoft Sentinel Automation Contributor role to Azure Security Insights (service principal) which gives Microsoft Sentinel the permission to trigger any playbook in that resource group. However, users with the Microsoft Sentinel Contributor role can also run the playbooks manually on security incidents.

![image](Images/Playbk.perm.png)

## Testing of the Playbook

The Sales-user account(created alongside the Sales-VM) was used to logon to the sales-VM via Azure Bastion with the trusted IP. Then a new user account (Random-user) was created  on the Sales-VM and it was added to the remote desktop users group to allow remote logon with Random-user account. 

![image](Images/CR.User.png)

Note: The new user account was only created to effectively demonstrate the testing of the playbook.

![image](Images/Random_user.png)

To simulate an attack scenario, the Random-user account was used to logon to the Sales-VM remotely via Azure Bastion but with an untrusted IP address. 

![image](Images/LogontoVM.png)

![image](Images/ConnecttoVM.png)

The detection of the usage of an untrusted IP for connection to the Sales-VM led to the firing of a Microsoft Sentinel alert and an incident thereafter. 

![image](Images/INCIDENT2.png)

While reviewing the incident page, it was observed that the automation rule had assigned the incident to a member in the SecOps team.

Further review of the incident page indicated that the playbook was succesfully triggered by the incident.

![image](Images/Ran_success.png)

Following the running of the playbook, it expected results were confirmed as follows: 

creation of a NSG security rule to block all inbound RDP connection to the Sales-VM;

![image](Images/NSG_success.png)

closing the remote session of the attacker (Random-user) that connected with an untrusted IP; 

![image](Images/Closed_success.png)

and sending an email notification to relevant members of the SecOps team for necessary actions.

![image](Images/email.png)

The recovery operations and incident management includes the following:

deleting the NSG rule created by the playbook action to allow remote connection to the VM;

![image](Images/NSG_Delete.png)
  
classifying the security incident;

![image](Images/manage_incident.png)
  
and closing of the incident.

![image](Images/resolved.png)

![image](Images/evidence_resolved.png)

## Further Tests

Further tests were implemented to assess the performance of the playbook. In this case, another user account (SecondRandom-user) was created on the Sales-VM and added to remote desktop users group. Afterwards, both user accounts (Random-user & SecondRandom-user) were logged-on to sales-VM via Azure Bastion with both users using different untrusted IPs. 

![image](Images/extra3.png)

![image](Images/IPs.png)

![image](Images/Users.png)

Similarly, the attack was detected, and it led to a security incident. The incident page dispalys the Accounts and IPs involved in the attack. Each of the attacker's sessions were also closed, and all the entities related to the attack were extracted and sent with the email notification. In cases where the attack is simultaneous, and involving multiple accounts and IPs, every entity will be extracted by the playbook just as seen in the incident page, and every of the attacker's sessions will be closed.

![image](Images/email2.png)

## Conclusion

This project succesfully demonstrates the implementation of playbooks in supporting incident response while enhancing the management of security operations. It also provides valuable insights into designing and orchestrating workflows in Azure Logic Apps for seamless integration with Microsoft Sentinel. Overall, the project highlights the end-to-end management of real-world security incidents covering the complete incident lifecycle from detection and containment to recovery and closure.

## Past Project

* Deployment of Azure Firewall for secure access and traffic control https://rhosinjay-cyb.github.io/Azure-Firewall/
* Deployment of Microsoft Sentinel to support cloud workload protection https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

