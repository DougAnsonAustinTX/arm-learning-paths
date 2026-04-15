---
title: Testing the installed/optimized JVM for PAC/BTI enablement
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---


### Setup the test script

Clone this repo and set execute permissions for the downloaded test script:

```bash
git clone https://github.com/DougAnsonAustinTx/pac-bti-jdk-assets 
cd pac-bti-jdk-assets
chmod 755 ./test-pacbti.sh
```

### Run the test script

Run the test script to confirm PAC/BTI enablement:

```bash
$HOME/test-pacbti.sh --java /usr/bin/java
```

Output should resemble:

```output
== JVM PAC/BTI check ==
java executable : /home/ubuntu/jdk/build/linux-aarch64-server-release/jdk/bin/java
libjvm          : /home/ubuntu/jdk/build/linux-aarch64-server-release/jdk/lib/server/libjvm.so

-- Host support (auxv/hwcaps) --
PAC APIA/generic support : yes (PACA=yes PACG=yes)
BTI support              : yes

-- JVM support/config --
UseBranchProtection flag : yes
JVM default flag value   : {default}

-- Binary instruction scan --
java contains PAC instr  : yes
java contains BTI instr  : yes
libjvm contains PAC instr: no
libjvm contains BTI instr: no

-- Verdict --
PAC status               : possible-but-not-proven
BTI status               : yes

Interpretation:
  PAC-capable build detected, but no running JVM cmdline was checked.
  BTI is likely enabled in the runtime binaries.
```

### What we have learned

By default, some OpenJDK JVMs are not published with PAC/BTI enabled.  This is primarily due to the number of older arm architectures running in the wild.  Over time, this will change though. If needed though, it's simple to create a private instance of a JVM that is built with PAC/BTI and registered with the underlying operating system.

Using JVMs that have the PAC/BTI features enabled, when run on platforms that support PAC/BTI, provide added safety and protection for Java-based workloads running in the JVM.
