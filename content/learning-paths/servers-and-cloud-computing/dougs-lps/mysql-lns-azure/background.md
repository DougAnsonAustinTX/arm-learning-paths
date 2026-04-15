---
title: Overview of Azure Cobalt 100 VM and MySQL Migrations

weight: 2

layout: "learningpathall"
---

### Azure Cobalt 100 Arm-based processor

Azure’s Cobalt 100 is Microsoft’s first-generation, in-house Arm-based processor. Built on Arm Neoverse N2, Cobalt 100 is a 64-bit CPU that delivers strong performance and energy efficiency for cloud-native, scale-out Linux workloads such as web and application servers, data analytics, open-source databases, and caching systems. Running at 3.4 GHz, Cobalt 100 allocates a dedicated physical core for each vCPU, which helps ensure consistent and predictable performance.

To learn more, see the Microsoft blog [Announcing the preview of new Azure VMs based on the Azure Cobalt 100 processor](https://techcommunity.microsoft.com/blog/azurecompute/announcing-the-preview-of-new-azure-vms-based-on-the-azure-cobalt-100-processor/4146353).

### MySQL Migrations

MySQL is a cross-platform relational database system whose storage engines and on-disk formats are designed for reliability and portability, but when moving between different CPU architectures such as x64 to Arm, Oracle’s MySQL documentation recommends a logical migration rather than copying raw data files directly: use mysqldump to export the database as SQL, transfer the dump to the Arm server, and import it with the mysql client on the target system. In the MySQL 8.4 Reference Manual, the “Copying MySQL Databases to Another Machine” section explicitly says that for transfers between different architectures, use mysqldump to create SQL statements and then load them on the other machine, while the mysqldump documentation notes that the utility is intended for backup or transfer to another SQL server. For Linux installs, MySQL also recommends using Oracle-provided distributions/packages for the destination host so the Arm server is running a supported build before the import.


## What you've learned and what's next

Now that you have the background on the Azure Cobalt 100 processor, MySQL and MySQL's "lift-n-shift" recommendations from migrating a MySQL database from an X64 server to an ARM server, we'll utilize Terraform and bash scripting to "lift-n-shift" an example database from an "on-prem" instance that you have setup into an Azure Neoverse VM running MySQL.

Next, we'll create our simulated "on-prem" x64 server, as an Azure VM for sake of example, and prep the VM to become our "on-prem" environment.