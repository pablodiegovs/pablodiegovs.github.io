---
layout: post
title:  "Getting network device facts in multi-vendor environment"
description: Getting network device facts in multi-vendor environment (Cisco IOS and Huawei VRP)
date:   2022-05-10 16:37:36
categories: Python3 Napalm
---
The goal of this post and script is obtain a general information from a group of network devices (Hostname, IP address management, Vendor, Model, OS Version and Serial Number)

![texture theme preview](https://github.com/pablodiegovs/pablodiegovs.github.io/raw/main/assets/images/Napalm-facts.jpg)

For the parsing we'll use Napalm python library. The instructions for how to install Napalm are in the follow link -> [napalm_install][napalm_install]

By default, Napalm has the support for Cisco IOS parsing, but for Huawei VRP parsing we need to install aditional drivers for community development -> [napalm_community][napalm_community]

For this scenario we are going to use 3 network devices (2 with Cisco IOS and 1 with Huawei VRP), all of them have enable SSHv2. We built a yaml file data base (devices/test.yml) with general connect information for these group of network devices. Yaml file is displayed bellow>


```css
devices:
  Cisco-IOS-Device-01:
    connections:
      cli:
        ip: 10.10.10.1
        protocol: ssh -o KexAlgorithms=diffie-hellman-group14-sha1
    os: ios
  Cisco-IOS-Device-02:
    connections:
      cli:
        ip: 10.10.10.2
        protocol: ssh -o KexAlgorithms=diffie-hellman-group14-sha1
    os: ios
  Huawei-VRP-Device-01:
    connections:
      cli:
        ip: 172.16.30.30
        protocol: ssh -o KexAlgorithms=diffie-hellman-group14-sha1
    os: huawei_vrp
```

On the python script it's necessary to import the following libraries, including napalm packages:

```javascript
import napalm
import yaml
import getpass
from tabulate import tabulate
```

Then, we're going to define a function where it contains the following sections described on the comments:

```javascript

def main():
    ### Get credentials for logging
    user = input("Username: ")
    pasw = getpass.getpass(prompt="Password: ")
    print("#"*100 + "\n")
    
    ### Select area o section to deploy script, in these case we are going to select TEST
    section = input("Select one department (IT/Financial/TEST): ")
    print("#"*100 + "\n")
    
    ### Associate vendor type with napalm driver
    driver_ios = napalm.get_network_driver("ios")
    driver_hvrp = napalm.get_network_driver("huawei_vrp")
    
    ### Pull devices parameters from yaml file to list
    with open('devices/{}.yml'.format(anillo)) as file:
        device_yaml = yaml.load(file, Loader=yaml.FullLoader)
    device_list = device_yaml['devices'].keys()
    
    ### Setting connect parameters for every vendor device
    network_devices = []
    for device in device_list:
        if device_yaml['devices'][device]['os'] == 'ios':
            network_devices.append(
                            driver_ios(
                                hostname = device_yaml['devices'][device]['connections']['cli']['ip'],
                                username = user,
                                password = pasw
                            )
            )
        elif device_yaml['devices'][device]['os'] == 'huawei_vrp':
            network_devices.append(
                            driver_hvrp(
                                hostname = device_yaml['devices'][device]['connections']['cli']['ip'],
                                username = user,
                                password = pasw
                            )
            )
    
    ### Variable header for table presentation
    devices_table = [["HOSTNAME", "IP-GESTION", "VENDOR", "MODEL", "OS_VERSION", "SERIAL_NUMBER"]]  # Variable header for table presentation
    
    ### Getting device facts
    for device in network_devices:
        print("Connect to {} ...".format(device.hostname))
        device.open()

        print("Getting Facts...")
        device_facts = device.get_facts()
        
        ### Associate parsing to table
        
        devices_table.append([device_facts["hostname"],
                                device.hostname,
                                device_facts["vendor"],
                                device_facts["model"],
                                device_facts["os_version"],
                                device_facts["serial_number"]
                                ])
        device.close()
    
    ### Display result
    print("#"*100 + "\n")
    print(tabulate(devices_table, headers="firstrow"))
    print("\n"+"#"*100 + "\n")

if __name__ == '__main__':
    main()

```

I hope that the script was helpfully.

[napalm_install]: https://napalm.readthedocs.io/en/latest/
[napalm_community]: https://napalm.readthedocs.io/en/latest/contributing/drivers.html
