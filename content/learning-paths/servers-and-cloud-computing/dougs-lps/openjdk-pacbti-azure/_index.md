---
title: Create custom OpenJDK JVM with PAC/BTI enabled on Azure Cobalt 100 processors 

draft: true
cascade:
    draft: true
    
minutes_to_complete: 30   

who_is_this_for: This is an introductory topic about creating and deploying an OpenJDK JVM, with the arm64 PAC/BTI features enabled, on Microsoft Azure Cobalt 100 Arm-based virtual machines. It is designed for developers using Java applications wanting to enable their Java VMs with the added security strength provided by the PAC/BTI instructions within the Arm64 v9 architecture.

learning_objectives: 
    - Provision an Azure Arm-based Cobalt 100 virtual machine using Azure console, with Ubuntu Pro 24.04 LTS as the base image
    - Download, compile and deploy the OpenJDK Java JVM on the Azure Arm64 virtual machine with PAC/BTI enabled
    - Confirm that the deployed JVM, on Cobalt 100, has the PAC/BTI features enabled and ready for use by the JVM. 

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
    - Java

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
      title: OpenJDK 
      link: https://openjdk.org/guide/ 
      type: documentation
  - resource:
      title: Arm64 v9 PAC instruction
      link: https://developer.arm.com/documentation/100076/0100/A64-Instruction-Set-Reference/A64-General-Instructions/PACGA?lang=en 
      type: documentation
  - resource:
      title: Arm64 v9 BTI instruction
      link: https://developer.arm.com/documentation/100076/0100/A64-Instruction-Set-Reference/A64-General-Instructions/BTI
      type: documentation


### FIXED, DO NOT MODIFY
# ================================================================================
weight: 1                       # _index.md always has weight of 1 to order correctly
layout: "learningpathall"       # All files under learning paths have this same wrapper
learning_path_main_page: "yes"  # This should be surfaced when looking for related content. Only set for _index.md of learning path content.
---
