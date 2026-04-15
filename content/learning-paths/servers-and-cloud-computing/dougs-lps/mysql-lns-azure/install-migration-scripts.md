---
title: Perform the the "Lift-n-Shift" migration into Azure Cloud

weight: 5

layout: "learningpathall"
---

### Introduction

In this section we will download and install our Lift-n-Shift migration script and perform the lift-n-shift from our "on-prem" X64 instance into the arm-based Azure Cloud VM. 

### Download the lift-n-shift script set

Open a SSH shell into your "on-prem" instance and go to our lift-n-shift assets previously downloaded:

```bash
cd $HOME/lift-n-shift-assets
```

### Configure the lift-n-shift script

Run the following script to create an SSH public key (just hit return for the password...):

```bash
scripts/create_ssh_key.sh
```

You should get a key that looks something like this:

```output
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDczfgUuS4pnNSXnNeK4lRR+CcmxCH+/Vl+aP9dhtGYX7DEVrWrKy9Hq7AKs3y1AUyW85MGBJEkgy8WQZui6T92UNZz+MGVoKm/++SyO/
....
tH33RAoBg2q3mJJgOCqCQ8k5kfAww== azureuser@YOUR_X64_ON_PREM_HOST
```

Save this key off. You will use it in the next step

Create a copy of the terraform.tfvars.example to terraform.tfvars:

```bash
cp terraform.tfvars.example terraform.tfvars
```

In an editor, open "terraform.tfvars" and make the following edits:

```edit
location              = "eastus2"
prefix                = "mysql-migrate"

source_mysql_database = "testdb"

admin_username        = "azureadmin"
ssh_public_key        = "RSA_KEY_GOES_HERE"

allowed_ssh_cidr      = "YOUR_PUBLIC_IP/32"
allowed_mysql_cidr    = "YOUR_PUBLIC_IP/32"

allow_mysql_inbound = true
vm_size         = "Standard_E4pds_v6"
os_disk_size_gb = 48

tags = {
  environment = "prod"
  application = "testdb-migration"
  owner       = "admin"
  managed_by  = "terraform"
}
```

You will want to edit:
1). the location (region) with the proper region you wish to place your Azure Arm-based Cloud VM into (above is set to "EastUS 2" region)
2). update the prefix to suite (Azure resources will be prepended with this..)
3). replace "RSA_KEY_GOES_HERE" with the RSA key you created above
4). replace "YOUR_PUBLIC_IP" with the public IP address of your "on-prem" x64 instance

Everything else can be left alone. 

### Log into Azure with the CLI

On the "on-prem" SSH shell, type:

```bash
az login
```

And follow the prompts to log into your Azure account. 

### Start the migration

On the "on-prem" SSH shell, type:

```bash
scripts/migrate.sh testdb
```

This will begin the migration of the "testdb" local DB into a newly created Arm-based (E32dps_v6) VM running in Azure Cloud. 

Assuming no errors in the tfvars config file, eventually you will receive a prompt:

```output
Migrating local DB: testdb to Cloud...
Creating Cloud VM...
Cloud VM created.
Backing up the local DB testdb...
Enter password:
```

Please supply the password that you provided when you created your "admin" MySQL user on your "on-prem" MySQL instance ("SuperStrongPassword") from the last section. You'll then be prompted again:

```output
Migrating local DB: testdb to Cloud...
Creating Cloud VM...
Cloud VM created.
Backing up the local DB testdb...
Enter password: 
Restoring local DB testdb onto the Cloud VM: Admin: admin IP: 20.98.229.225
You will need to open a second window and ssh into the VM and then look in /root for the password
Enter password:
```

In order to acquire this password, you must:

        1. Open another shell into your "on-prem" X64 instance. 
        2. From the new shell in you "on-prem" X64 instance, look in $HOME/.ssh.  There should be a file ending in "rsa". Note the filename. 
        3. In the Azure Console, under "Virtual Machines", locate your new Arm-based Cloud VM (it should be prefixed per the prefix setting in the terraform.tfvars file)
        4. Note the public IP address of your new Arm-based Cloud VM
        5. Back on the second shell into your "on-prem" x64 server, type the following replacing YOUR_RSA_FILENAME and YOUR_ARM_BASED_VM_PUBLIC_IP_ADDRESS from what you saved above:


```bash
ssh -i $HOME/.ssh/YOUR_RSA_FILENAME azureadmin@YOUR_ARM_BASED_VM_PUBLIC_IP_ADDRESS
```

You should now have another shell into your Arm-based Azure VM. Within that shell, type the following:

```bash
sudo su - 
cat /root/mysql_root_password.txt
```

That's the required password. Supply that password to your first SSH session on the "on-prem" host and the script should complete:

```output
Migrating local DB: testdb to Cloud...
Creating Cloud VM...
Cloud VM created.
Backing up the local DB testdb...
Enter password: 
Restoring local DB testdb onto the Cloud VM: Admin: admin IP: 20.98.229.225
You will need to open a second window and ssh into the VM and then look in /root for the password
Enter password: 
```

{{% notice Note %}}
The original script may time out waiting for this second password. If it does, right in that same shell, type the following replacing YOUR_ARM_BASED_VM_PUBLIC_IP_ADDRESS with the public IP address of your Arm-based VM:

```bash
 gunzip -c testdb.sql.gz | mysql -h YOUR_ARM_BASED_VM_PUBLIC_IP_ADDRESS -u admin -p 
```

You will be prompted again for the second password that you have retrieved previously... provide that and the migration will complete. 
{{% /notice %}}

At this point, your "on-prem" MySQL database (testdb) has been successfully migrated into a new Arm-based VM in Azure Cloud. 

### What we learned and what's next

This script and proceedure is suitable for migrating specific MySQL databases from "on-prem" X64 environments into Arm-based Azure Cloud VM instances. Our "testdb" DB has been migrated and is ready for use by sysbench in the Azure Cloud.  Next, we'll run the sysbench benchmarking to gauge any performance differences attained by migrating the "on-prem" instance into Azure Cloud. 