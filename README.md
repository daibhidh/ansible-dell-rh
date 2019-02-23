ansible-dell-rh
=========

This role is for installing the Dell OMSA and dell-upgrade software.  IT also handles custome disk configurations if the variable dell_disk_sets is defined.  See the Storage section below for more information on this.


Role Variables
--------------
This role contains two static variables defined in vars/main.yml which are 'repo_command' and 'packages' respectively.  Repo command is what command should be run for installing the dell omsa repositories where packages is what dell packages should be installed.

In order for the system to be 'bootstrapped' (Install OMSA Repo) the first time it is run the variable dell_install must be defined.  This can be done either in the playbook, host_vars/group_vars, or more optimally with ```--extra-vars="dell_install=true"``` the first time you run the playbook/role on a system.  


Storage
-------

In order to setup/ensure a custom disk configuration the 'tasks/storage.yml' set of tasks can be run given the the dell_disk_sets variable is defined.  This is not defined by default.  This functionailty also makes use of the 'vars/storage_spec.yml' file in order to properly parse and use the output from the omreport command (XML output).

The first and mandotory portion to be defined is:

```
---
dell_disk_sets:
  docker:
    raid: r1
    physical_disks: 0:1:0,0:1:1
    vgroup: docker

  vmstore:
    raid: r1
    physical_disks: 0:1:2,0:1:3
...
```

The above example would configure two seperate Raid arrays and create a logical volume group named docker for the 'docker' aid group.  If a volume group is not required/wanted then not defining the 'vgroup' for that set will skip that task for that disk.

The raid variable specifies what raid level to use for the raid, and the physical_disks is a comma seperated list of disks to be used.  Currently controller 0 is being assumed.  The disk specifications follow the same format as used by the omconfig command (which is what gets run).


Additional variable sections can be defined to create logical volumes as well as setup the filesystems and mount them/add to fstab as shown below.

```
---
dell_lgvs:
  - name: opt
    vg: docker
    size: 100G

dell_filesystems:
  docker:
    fstype: xfs
    device: /dev/mapper/docker-opt
    mount_path: /opt/docker
    owner: docker_user
    group: docker
    mode: 0755
...
```

The first variable section in the above example would create a thinprovisioned logical volume on the docker vg from earliernamed opt and with a max size of 100G.  Multiple logical volumes can be specifed following the list format shown in the example.

The second variable above is used for creating filesystems, mountpoints, and correcting permissions if desired.  This follows the dictionary format for ease of looping.
If you are not wanting the filesystem to be mounted then exclude the mount_path, owner, group and mode portions of the dict.

If special filepermsions (owner, group, or mode) are not needing to be defined for mounted filesystems then do not define the owner, group and mode portions.


Required variables for the storage section can be summarized as the following:

dell_disk_sets  =  Must be defined for the storage tasks to run.  Only vgroup is optional.
dell_lgvs  =  Must be defined for creation of logical volumes.  No optional values in list.
dell_filesystems  =  Must be defined for filesystem creation, mounting and permissions setting.  mount_path must be defined for mounting/fstab as well as permissions setting.  owner, group and mode must be defined if permission setting is desired.
