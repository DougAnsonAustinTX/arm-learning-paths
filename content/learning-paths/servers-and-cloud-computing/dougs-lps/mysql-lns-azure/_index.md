---
title: Lift-n-Shift MySQL from on-prem to Azure Cloud Cobalt 100 VM 

draft: true
cascade:
    draft: true

minutes_to_complete: 30   

who_is_this_for: This is an introductory topic about the "Lift-n-Shift" of MySQL from an "on-premise" installation (x86) to the Microsoft Azure Cobalt 100 Arm-based virtual machines. It is designed for developers migrating MySQL from x86_64 on-premise to Arm architecture in the Azure cloud.

learning_objectives: 
    - Provision an Azure Arm-based Cobalt 100 virtual machine using a Terraform script and the Azure CLI, with Ubuntu Pro 24.04 LTS as the base image
    - On the "on-premise" host, performing the actual manual "lift-n-shift" of a sysbench-based MySQL database to the Cobalt 100 VM. 
    - Perform sysbench testing and comparative benchmarking between the on-premise host and the Arm64 Cobalt 100 virtual machine

prerequisites:
    - A [Microsoft Azure](https://azure.microsoft.com/) account with access to Cobalt 100 based instances (Dpsv6)


author: Doug Anson

### Tags
skilllevels: Introductory
subjects: Performance and Architecture
cloud_service_providers:
  - Microsoft Azure

armips:
    - Neoverse

tools_software_languages:
    - MySQL

operatingsystems:
    - Linux

further_reading:
  - resource:
      title: Azure Virtual Machines documentation
      link: https://learn.microsoft.com/en-us/azure/virtual-machines/
      type: documentation
  - resource:
      title: Azure Container Instances documentation
      link: https://learn.microsoft.com/en-us/azure/container-instances/
      type: documentation
  - resource:
      title: MySQL 
      link: https://dev.mysql.com/doc/ 
      type: documentation
  - resource:
      title: sysbench benchmarking tools for MySQL
      link: https://manpages.ubuntu.com/manpages/trusty/man1/sysbench.1.html 
      type: documentation


### FIXED, DO NOT MODIFY
# ================================================================================
weight: 1                       # _index.md always has weight of 1 to order correctly
layout: "learningpathall"       # All files under learning paths have this same wrapper
learning_path_main_page: "yes"  # This should be surfaced when looking for related content. Only set for _index.md of learning path content.
---
