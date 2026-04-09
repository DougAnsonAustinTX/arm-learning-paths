---
title: Test arm PAC/BTI instruction readiness using an AWS Greengrass custom component

draft: true
cascade:
    draft: true
    
minutes_to_complete: 30   

who_is_this_for: This is an introductory topic about utilizing AWS IoT Greengrass to create and deploy a custom component that will test for PAC/BTI readiness on arm platforms.

learning_objectives: 
    - Create a custom AWS IoT Greengrass component with the PAC/BTI testharness
    - Install AWS IoT Greengrass onto arm v8 (i.e. RPi5) and arm v9 (i.e. Jetson Thor or similar) platforms
    - Perform PAC/BTI checks via the custom greengrass component to confirm PAC/BTI presence and readiness

prerequisites:
    - A [Amazon AWS](https://aws.amazon.com/) account with access to AWS IoT Greengrass and AWS S3


author: Varun Chari, Doug Anson

### Tags
skilllevels: Introductory
subjects: Performance and Architecture
cloud_service_providers:
  - Microsoft Azure

armips:
    - Neoverse

tools_software_languages:
    - Java
    - YAML

operatingsystems:
    - Linux

further_reading:
  - resource:
      title: AWS IoT Greengrass documentation
      link: https://aws.amazon.com/greengrass/
      type: documentation


### FIXED, DO NOT MODIFY
# ================================================================================
weight: 1                       # _index.md always has weight of 1 to order correctly
layout: "learningpathall"       # All files under learning paths have this same wrapper
learning_path_main_page: "yes"  # This should be surfaced when looking for related content. Only set for _index.md of learning path content.
---
