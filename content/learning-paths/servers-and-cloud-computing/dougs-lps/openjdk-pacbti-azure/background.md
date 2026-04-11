---
title: Overview of Azure Cobalt 100, OpenJDK, and the PAC and BTI Arm v9 instructions

weight: 2

layout: "learningpathall"
---

## Azure Cobalt 100 Arm-based processor

Azure’s Cobalt 100 is Microsoft’s first-generation, in-house Arm-based processor. Built on Arm Neoverse N2, Cobalt 100 is a 64-bit CPU that delivers strong performance and energy efficiency for cloud-native, scale-out Linux workloads such as web and application servers, data analytics, open-source databases, and caching systems. Running at 3.4 GHz, Cobalt 100 allocates a dedicated physical core for each vCPU, which helps ensure consistent and predictable performance.

To learn more, see the Microsoft blog [Announcing the preview of new Azure VMs based on the Azure Cobalt 100 processor](https://techcommunity.microsoft.com/blog/azurecompute/announcing-the-preview-of-new-azure-vms-based-on-the-azure-cobalt-100-processor/4146353).

## OpenJDK and the PAC/BTI Arm v9 instructions

OpenJDK is the open-source reference implementation of the Java Platform, Standard Edition (Java SE). It provides the core Java compiler, runtime, and class libraries that developers use to build and run Java applications across many operating systems and hardware platforms. Originally released by Sun Microsystems and now primarily stewarded by Oracle with broad community and industry participation, OpenJDK forms the basis for most modern Java distributions. In practice, it gives organizations a common, standards-based Java foundation while allowing different vendors to package, support, and optimize their own builds.

Armv9 Pointer Authentication (PAC) and Branch Target Identification (BTI) are security features designed to make control-flow attacks harder. PAC helps protect return addresses and pointers by adding a cryptographic signature that is checked before the pointer is used, which can detect tampering such as return-oriented programming attempts. BTI complements this by restricting where indirect branches are allowed to land, helping prevent attackers from jumping into unintended instruction sequences. Together, PAC and BTI strengthen software defenses at the instruction-set level, especially for modern operating systems, hypervisors, and applications that need improved resistance to memory-corruption exploits.

## What you've learned and what's next

Now that you have the background on the Azure Cobalt 100 processor and the incorporation of PAC/BTI support in the OpenJDK source base, you'll create the virtual machine that will enable you to build the OpenJDK JVM with PAC/BTI support and you will be able to associate that JVM to enable it to be used in Java workloads on that VM. Doing so will provide PAC/BTI support to those Java workloads.
