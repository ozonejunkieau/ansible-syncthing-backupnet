# ansible-syncthing-backupnet
Ansible plays for secure deployment of a Syncthing network, including disk encryption and Syncthing Discovery servers.

Blog post and improved doco coming.

# Inventory

```
  syncthing_discovery_servers:
    hosts:
      syncthing-disco.example.com:

  syncthing_servers:
    hosts:
      town-a.example.com:
        ansible_host: 172.17.2.103
        syncthing_disks:
          - disk_id: scsi-112233aabbcc
            name: disk_name_1
          - disk_id: scsi-112233aabbcc_DDEEFF
            name: disk_name_2
          - disk_id: scsi-112233aabbcc+DDEFFF
            name: disk_name_3

      city-a.syncthing.example.com:
        ansible_host: 172.17.2.104
        syncthing_disks:
          - disk_id: scsi-112233aabbcc_DDEEFFA
            name: disk_name_1
          - disk_id: scsi-112233aabbcc_DDEEFFB
            name: disk_name_2

```