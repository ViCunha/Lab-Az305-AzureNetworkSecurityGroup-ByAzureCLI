
### Overview
---
Configure a public access to a Azure Virtual Machine (VM)
   
### Key Aspects
---
- Azure Portal
- Azure Virtual Machine (VM)
- Azure Network Security Group (NSG)

### Environment
---
Microsoft Azure Portal
- Valid Subscription
- Sandbox

### Actions
---
Create a Linux virtual machine and install Nginx
  - From Cloud Shell, run the following az vm create command to create a Linux VM
```
az vm create \
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--name my-vm \
--public-ip-sku Standard \
--image Ubuntu2204 \
--admin-username azureuser \
--generate-ssh-keys
```

- Run the following az vm extension set command to configure Nginx on your VM
```
az vm extension set \
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--vm-name my-vm \
--name customScript \
--publisher Microsoft.Azure.Extensions \
--version 2.1 \
--settings '{"fileUris":["https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh"]}' --protected-settings '{"commandToExecute": "./configure-nginx.sh"}'
```


Access your web server
- Run the following az vm list-ip-addresses command to get your VM's IP address and store the result as a Bash variable
```
IPADDRESS="$(az vm list-ip-addresses \
 --resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--name my-vm \
--query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" --output tsv)"
```

- Run the following curl command to download the home page
```
curl --connect-timeout 5 http://$IPADDRESS
```

- Run the following to print your VM's IP address to the console
```
echo $IPADDRESS
```


List the current network security group rules

- Run the following az network nsg list command to list the network security groups that are associated with your VMTasks
```
az network nsg list \ 
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--query '[].name' --output tsv
```

- Run the following az network nsg rule list command to list the rules associated with the NSG named my-vmNSG
```
az network nsg rule list \ 
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--nsg-name my-vmNSG
```

- Run the az network nsg rule list command a second time
```
az network nsg rule list \
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--nsg-name my-vmNSG \
--query '[].{Name:name, Priority:priority, Port:destinationPortRange, Access:access}' \
--output table
```

Create the network security rule
- Run the following az network nsg rule create command to create a rule called allow-http that allows inbound access on port 80
```
az network nsg rule create \
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--nsg-name my-vmNSG \
--name allow-http \
--protocol tcp \
--priority 100 \
--destination-port-range 80 \
--access Allow
```

- To verify the configuration, run az network nsg rule list to see the updated list of rules
```
az network nsg rule list \
--resource-group "learn-ef5bf4e8-801f-499a-a697-63b7da6d6ff9" \
--nsg-name my-vmNSG \
--query '[].{Name:name, Priority:priority, Port:destinationPortRange, Access:access}' \
--output table
```

Access your web server again
- Run the same curl command that you ran earlier
```
curl --connect-timeout 5 http://$IPADDRESS
```

### Media
---
![image](https://github.com/user-attachments/assets/5e315476-7fda-4177-946d-87ca54e8d50a)
---
![image](https://github.com/user-attachments/assets/3486d574-32bb-46c3-a6e4-913da51f826c)


### References
---
- [Configure network access](https://learn.microsoft.com/en-us/training/modules/describe-azure-compute-networking-services/9-exercise-configure-network-access)
