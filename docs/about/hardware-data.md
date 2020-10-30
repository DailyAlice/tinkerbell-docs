---
title: Hardware Data
date: 2020-07-29
---

# Hardware Data

Tinkerbell uses hardware data to identify the hardware targeted by a Workflow. Typically, it will be used to describe the hardware of a Worker, and can include network interfaces, storage disks, and file systems. Hardware data is JSON formatted, and stored on the Provisioner in PostgreSQL.

While the hardware data is essential, not all the properties are required for every Worker or workflow. Hardware data is intended to be of use to the developer/hardware owner instead of a rigid, explicit definition in service of Tinkerbell. You can include more data in hardware data than is touched by your workflow; it can be as detailed as is useful to you.

## A Minimal Hardware Data Example

While not all fields are required in hardware data, there are some requirements in content and structure. When creating hardware data, it might be useful to start with the minimal data given below and add the properties you would want to use in your workflow. Be sure to change the values of the fields to match your environment.

```json
{
  "id": "0eba0bf8-3772-4b4a-ab9f-6ebe93b90a94",
  "metadata": {
    "facility": {
      "facility_code": "onprem"
    },
    "instance": {},
    "state": ""
  },
  "network": {
    "interfaces": [
      {
        "dhcp": {
          "arch": "x86_64",
          "ip": {
            "address": "192.168.1.5",
            "gateway": "192.168.1.1",
            "netmask": "255.255.255.248"
          },
          "mac": "00:00:00:00:00:00",
          "uefi": false
        },
        "netboot": {
          "allow_pxe": true,
          "allow_workflow": true
        }
      }
    ]
  }
}
```
A few details on the fields:
 
- The `id` field is a unique UUID that you provide to identify this specific hardware data. It is the only required field.
- The `network.interfaces[].netboot.allow_workflow` is set to true to allow your Worker boot into workflow mode.
- The `network.interfaces[].dhcp.ip.address` is the IP Address that Boots will give to your Worker, and it can also be used to identify the hardware data in the database. A Worker machine can be identified by either IP Address or MAC Address at workflow creation and boot up.
- The `network.interfaces[].dhcp.mac` is the MAC Address for your Worker, and can used to identify the hardware data in the database. A Worker machine can be identified by either IP Address or MAC Address at workflow creation and boot up.

## Hardware Data CLI Commands
 
Hardware data is pushed to the database on the Provisioner with the [`tink hardware push`](/cli-reference/hardware/#tink-hardware-push) command, and deleted with [`tink hardware delete`](/cli-reference/hardware/#tink-hardware-delete).

Hardware data can be retrieved by ID, IP address, or MAC address with [`tink hardware id`](/cli-reference/hardware/#tink-hardware-id), [`tink hardware ip`](/cli-reference/hardware/#tink-hardware-ip), and [`tink hardware mac`](/cli-reference/hardware/#tink-hardware-mac), respectively. You can retrieve all hardware data in the database with [`tink hardware all`](/cli-reference/hardware/#tink-hardware-all).

A complete list of CLI commands and examples is in the [CLI Reference](/cli-reference/hardware/).

## A Robust Hardware Data Example

This example hardware data describes a Worker with a single network device on it, that lives in a Packet facility, and has many fields that might be used as part of an inventory management system.

```json
{
  "id": "58b0184c-1ba5-41d3-baec-5e453b2de717",
  "metadata": {
    "bonding_mode": 5,
    "custom": {
      "preinstalled_operating_system_version": {},
      "private_subnets": []
    },
    "facility": {
      "facility_code": "ewr1",
      "plan_slug": "c2.medium.x86",
      "plan_version_slug": ""
    },
    "instance": {
      "crypted_root_password": "redacted",
      "operating_system_version": {
        "distro": "ubuntu",
        "os_slug": "ubuntu_18_04",
        "version": "18.04"
      },
      "storage": {
        "disks": [
          {
            "device": "/dev/sda",
            "partitions": [
              {
                "label": "BIOS",
                "number": 1,
                "size": 4096
              },
              {
                "label": "SWAP",
                "number": 2,
                "size": 3993600
              },
              {
                "label": "ROOT",
                "number": 3,
                "size": 0
              }
            ],
            "wipe_table": true
          }
        ],
        "filesystems": [
          {
            "mount": {
              "create": {
                "options": ["-L", "ROOT"]
              },
              "device": "/dev/sda3",
              "format": "ext4",
              "point": "/"
            }
          },
          {
            "mount": {
              "create": {
                "options": ["-L", "SWAP"]
              },
              "device": "/dev/sda2",
              "format": "swap",
              "point": "none"
            }
          }
        ]
      }
    },
    "manufacturer": {
      "id": "",
      "slug": ""
    },
    "state": ""
  },
  "network": {
    "interfaces": [
      {
        "dhcp": {
          "arch": "x86_64",
          "hostname": "server001",
          "ip": {
            "address": "192.168.1.5",
            "gateway": "192.168.1.1",
            "netmask": "255.255.255.248"
          },
          "lease_time": 86400,
          "mac": "00:00:00:00:00:00",
          "name_servers": [],
          "time_servers": [],
          "uefi": false
        },
        "netboot": {
          "allow_pxe": true,
          "allow_workflow": true,
          "ipxe": {
            "contents": "#!ipxe",
            "url": "http://url/menu.ipxe"
          },
          "osie": {
            "base_url": "",
            "initrd": "",
            "kernel": "vmlinuz-x86_64"
          }
        }
      }
    ]
  }
}
```

A few details on some of the notable fields included in the above example:

- `metadata.instance.storage.disks[].wipe_table` is set to `true` to allow disk wipe.
- `metadata.facility.plan_slug` is a slug for the worker class. The value for this property depends on how you setup your workflow. While it is required if you are using the OS images from [packet-images](https://github.com/packethost/packet-images) repository, it may be left out if not used at all in the workflow.