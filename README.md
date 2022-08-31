# ansible-syncthing-backupnet
These Ansible plays are used for the deployment of a distributed network of hosts for offsite backups.

Broadly speaking, the requirement was an easy to manage solution that allowed for a small, low power PC to be located at friends or family and data is remotely synchronised to this host. Disk encryption was desired to lesser the impact of the hardware falling into the wrong hands.

Hosts are all connected via VPN, currently OpenVPN but likely Tailscale in the future. This is not done via Ansible at this time.

All disks used for storing backups are encrypted via LUKS, managed via keyfiles. The keyfiles are generated on the control node side and saved to a specific path, with permissions being as secured as possible. The idea being that this path is then backed up via multiple USB drives to ensure keys are not lost. The deployed hosts do not have keyfiles permanently stored on them, it is temporarily placed in a ramdisk for the unlocking to occur and then removed. This necessitates the redeployment of keys in the event of a reboot, which just consists of rerunning the Ansible plays.

A discovery server is deployed within the VPN to allow hosts to find each other without relying on public discovery. In fact, the usage of public discovery servers is disabled, as is local network discovery. This is to ensure that all required traffic is internal to the VPN instead of relying on external services.

Blog post with more details is coming.

## Operating System Selection
The plays have successfully been tested on Centos Stream 8, and will likely work without issue on Fedora. Some modifications will be required for usage on Debian/Ubuntu.

## Inventory & Host Variables
The following variables are expected to be defined for each node. Disks are access via the `disk-id` and are mapped into a path using the `name`.

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

## TLS Certificates
The plays assume the existence of SSL certificates in a specified path with the name of `{{ inventory_hostname }}.crt` and `{{ inventory_hostname }}.key`.

## Plays

### Syncthing Discovery Server (`play_syncthing_disco.yml`)

This play deploys a discovery server onto a host with a TLS certificate, systemd unit file and configures firewall access to allow access.

### Disk Encryption (`play_syncthing_disks.yml`)

This play takes care of the disks used for storing Syncthing data. It relies on a list of disks with the name and id defined, these are configured with LUKS encryption and then mounted via LVM.

_NOTE: This will destroy data on disks the first time it is run, be careful!_

### Syncthing (`play_syncthing_disks.yml`)

This play deploys the syncthing application with TLS, sets a username and password, configured firewall rules.