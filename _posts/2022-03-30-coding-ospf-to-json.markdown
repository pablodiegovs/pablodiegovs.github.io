---
layout: post
title:  "OSPF neighbors table to json dictionary"
description: Transform "show ip ospf neighbor" Cisco command to json dictionary for parsing output
date:   2022-03-30 16:37:36
categories: Python3 Netmiko TextFSM
---
We'll deploy python script using netmiko and textFSM libraries for convert the show command output to json dictionary, for parsing purpose.

![texture theme preview](https://github.com/pablodiegovs/pablodiegovs.github.io/raw/main/assets/images/OSPF-Netmiko-TextFSM.jpg)

Here's the link information that I used for build the following script shown in this github page. [python-neteng][python-neteng]

The scenario consist on extract OSPF Neighbor table from R1 that have four OSPF neighbors, the 'show ip ospf neighbor' displayed below:


```javascript

R1#show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
20.20.20.20      0   FULL/DROTHER    00:00:36    172.16.26.13  GigabitEthernet1/8
30.30.30.30      0   FULL/DROTHER    00:00:36    172.16.26.15  GigabitEthernet1/9
40.40.40.40      0   FULL/  -        00:00:34    172.16.26.17  GigabitEthernet1/7
50.50.50.50      0   FULL/  -        00:00:33    172.16.26.19  GigabitEthernet1/6
```

The library textFSM was developed by Google for output text parsing, and for correct use it's necessary create a template file specifying parameters and variables for analyze output router command. Underneath it's a template used for OSPF neighbor parsing and saved as template/show_ospf.template

```javascript

Value NeighborID (\S+)
Value Pri (\d+)
Value State (FULL/DROTHER|FULL/  -    )
Value DeadTime (..:..:..)
Value Add (\S+)
Value Interface (\S+)

Start
 ^${NeighborID}\s+${Pri}\s+${State}\s+${DeadTime}\s+${Add}+\s+${Interface} -> Record
```

The first part shown on the text above is the variable part, where we declare one by one all variables displayed on 'show ip ospf nei" output. The second part is about structure for the statement of each OSPF neighbor. For more information you can visit: [python-neteng][python-neteng]

Install the following libraries (this post doesn't include the procedure for install python libraries, you'd use pip install 'library-name' command):

```javascript

from netmiko import ConnectHandler
import textfsm
import json
```

Set connection parameters for the device that we need to collect OSPF neighbors table information, and connect it using netmiko connect handler. By default, we'll use SSH on port 22 and the secret password it's not neccesary for this device.

```javascript

device = {
    'device_type': 'cisco_ios',
    'host':   '10.10.10.10',
    'username': 'user_name',
    'password': 'private_password123',
}
net_connect = ConnectHandler(**device)
net_connect.enable()
```

Firstly, execute Cisco IOS command "show ip ospf neighbor" using send_command from netmiko and save on disp_command variable. Then, we'll use textFM method with template developed earlier and OSPF output command as parameters. The method 'ParseTexttoDicts' give us a list variable from "show ip ospf neighbor" command.

```javascript

disp_command = net_connect.send_command('show ip ospf neigh')
template = open('template/show_ospf.template')
re_table = textfsm.TextFSM(template)
header = re_table.header
result = re_table.ParseTextToDicts(disp_command)

```

Finally, we'll export the result to json file to save as backup and future uses or parsing purpose.

```javascript
ospf_save = open('backup/bckp_ospf.json','w')
ospf_save.write(json.dumps(result, indent=2))
ospf_save.close
template.close
print (result)
```

the OSPF neighbor JSON file will look like this: 

```javascript
[
  {
    "NeighborID": "20.20.20.20",
    "Pri": "0",
    "State": "FULL/DROTHER",
    "DeadTime": "00:00:34",
    "Add": "172.16.26.13",
    "Interface": "GigabitEthernet1/8"
  },
  {
    "NeighborID": "30.30.30.30",
    "Pri": "0",
    "State": "FULL/DROTHER",
    "DeadTime": "00:00:36",
    "Add": "172.16.26.15",
    "Interface": "GigabitEthernet1/9"
  },
  {
    "NeighborID": "40.40.40.40",
    "Pri": "0",
    "State": "FULL/  -    ",
    "DeadTime": "00:00:34",
    "Add": "172.16.26.17",
    "Interface": "GigabitEthernet1/7"
  },
  {
    "NeighborID": "50.50.50.50",
    "Pri": "0",
    "State": "FULL/  -    ",
    "DeadTime": "00:00:33",
    "Add": "172.16.26.19",
    "Interface": "GigabitEthernet1/6"
  },
]
```

I hope that the script was helpfully.

[python-neteng]: https://pyneng.readthedocs.io/en/latest/book/21_textfsm/README.html
