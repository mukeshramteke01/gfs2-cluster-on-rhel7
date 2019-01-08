### KVM fencing device configuration and installation

#### Install the required fencing packages on KVM host

    # yum install fence-virt fence-virtd fence-virtd-libvirt fence-virtd-multicast fence-virtd-serial
    

#### Create the new directory to store the fence key. Create the random key to use for fencing.

    # mkdir -p /etc/cluster

    # cd /etc/cluster

    # touch fence_xvm.key

    #dd if=/dev/urandom of=/etc/cluster/fence_xvm.key bs=4k count=1
    1+0 records in
    1+0 records out
    4096 bytes (4.1 kB, 4.0 KiB) copied, 0.000170187 s, 24.1 MB/s

    # ll /etc/cluster/fence_xvm.key 
    -rw-r--r--. 1 root root 4096 Oct  8 17:14 /etc/cluster/fence_xvm.key


#### create dir on cluster nodes

    # mkdir -p /etc/cluster 


#### Copy the fence keys to all cluster nodes

    # scp /etc/cluster/fence_xvm.key 192.168.100.51:/etc/cluster/fence_xvm.key
    root@192.168.100.51's password: 
    fence_xvm.key                                                                                                                                                                    100% 4096    74.3KB/s   00:00    

    # scp /etc/cluster/fence_xvm.key 192.168.100.52:/etc/cluster/fence_xvm.key
    root@192.168.100.52's password: 
    fence_xvm.key                                                                                                                                                                    100% 4096   274.4KB/s   00:00    

    # scp /etc/cluster/fence_xvm.key 192.168.100.53:/etc/cluster/fence_xvm.key
    root@192.168.100.53's password: 
    fence_xvm.key                                

#### “fence_virtd -c” command to create “/etc/fence_virt.conf” file

      # fence_virtd -c
      Module search path [/usr/lib64/fence-virt]: 

      Available backends:
          libvirt 0.3
      Available listeners:
          multicast 1.2
          serial 0.4

      Listener modules are responsible for accepting requests
      from fencing clients.

      Listener module [multicast]: 

      The multicast listener module is designed for use environments
      where the guests and hosts may communicate over a network using
      multicast.

      The multicast address is the address that a client will use to
      send fencing requests to fence_virtd.

      Multicast IP Address [225.0.0.12]: 

      Using ipv4 as family.

      Multicast IP Port [1229]: 

      Setting a preferred interface causes fence_virtd to listen only
      on that interface.  Normally, it listens on all interfaces.
      In environments where the virtual machines are using the host
      machine as a gateway, this *must* be set (typically to virbr0).
      Set to 'none' for no interface.

      Interface [virbr0]: virbr1

      The key file is the shared key information which is used to
      authenticate fencing requests.  The contents of this file must
      be distributed to each physical host and virtual machine within
      a cluster.

      Key File [/etc/cluster/fence_xvm.key]: 

      Backend modules are responsible for routing requests to
      the appropriate hypervisor or management layer.

      Backend module [libvirt]: 

      The libvirt backend module is designed for single desktops or
      servers.  Do not use in environments where virtual machines
      may be migrated between hosts.

      Libvirt URI [qemu:///system]: 

      Configuration complete.

      === Begin Configuration ===
      backends {
        libvirt {
          uri = "qemu:///system";
        }

      }

      listeners {
        multicast {
          port = "1229";
          family = "ipv4";
          interface = "virbr1";
          address = "225.0.0.12";
          key_file = "/etc/cluster/fence_xvm.key";
        }

      }

      fence_virtd {
        module_path = "/usr/lib64/fence-virt";
        backend = "libvirt";
        listener = "multicast";
      }

      === End Configuration ===
      Replace /etc/fence_virt.conf with the above [y/N]? y


#### Enable and start fence_virtd service

      # systemctl enable fence_virtd.service 
      Created symlink /etc/systemd/system/multi-user.target.wants/fence_virtd.service → /usr/lib/systemd/system/fence_virtd.service.

      # systemctl start fence_virtd.service 

      # systemctl status fence_virtd.service 
      ● fence_virtd.service - Fence-Virt system host daemon
         Loaded: loaded (/usr/lib/systemd/system/fence_virtd.service; enabled; vendor preset: disabled)
         Active: active (running) since Mon 2018-10-08 17:29:45 IST; 3s ago
        Process: 30958 ExecStart=/usr/sbin/fence_virtd $FENCE_VIRTD_ARGS (code=exited, status=0/SUCCESS)
       Main PID: 30959 (fence_virtd)
          Tasks: 1 (limit: 4915)
         Memory: 2.8M
         CGroup: /system.slice/fence_virtd.service
                 └─30959 /usr/sbin/fence_virtd -w

      Oct 08 17:29:45 mukesh.linjb.com systemd[1]: Starting Fence-Virt system host daemon...
      Oct 08 17:29:45 mukesh.linjb.com systemd[1]: Started Fence-Virt system host daemon.
      Oct 08 17:29:45 mukesh.linjb.com fence_virtd[30959]: fence_virtd starting.  Listener: libvirt  Backend: multicast

#### Allow the port

        # firewall-cmd --permanent --add-port=1229/tcp
        success
        # firewall-cmd --reload
        success


Run on cluster nodes
========================

#### Install fence-virt package on all the server.

      # yum install fence-virt

#### The following commands much be scuesscced in an order to configure the fencing in cluster

      # fence_xvm -o list
        iscsi-storage                    7c860c2f-a1a9-4dfd-adbf-bcfe07606ad7 on
        node-1                           2caa20bc-4223-4568-a8a9-cdac1c31a60d on
        node-2                           976d56b1-c4e2-4a0b-b5bf-b86469126efe on
        node-3                           94a836b0-6386-4010-9e78-ec1993192784 on
        
#### Check below output  

        [root@node1 ~]# pcs stonith show
        NO stonith devices configured

        [root@node1 ~]# pcs status
        Cluster name: mycluster
        WARNING: no stonith devices and stonith-enabled is not false
        Stack: corosync
        Current DC: node1.linjb.com (version 1.1.16-12.el7-94ff4df) - partition with quorum
        Last updated: Mon Oct  8 18:27:01 2018
        Last change: Mon Oct  8 18:03:57 2018 by root via cibadmin on node1.linjb.com

        3 nodes configured
        6 resources configured

        Online: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

        Full list of resources:

         Clone Set: dlm-clone [dlm]
             Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
         Clone Set: clvmd-clone [clvmd]
             Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

        Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled

#### Configure fence_xvm fence agent on pacemaker cluster

        # pcs stonith create xvmfence fence_xvm key_file=/etc/cluster/fence_xvm.key

        # pcs stonith 
          xvmfence	(stonith:fence_xvm):	Started node1.linjb.com

        # pcs stonith --full
         Resource: xvmfence (class=stonith type=fence_xvm)
          Attributes: key_file=/etc/cluster/fence_xvm.key
          Operations: monitor interval=60s (xvmfence-monitor-interval-60s)

        # fence_xvm -o list
          iscsi-storage                    7c860c2f-a1a9-4dfd-adbf-bcfe07606ad7 on
          node-1                           2caa20bc-4223-4568-a8a9-cdac1c31a60d on
          node-2                           976d56b1-c4e2-4a0b-b5bf-b86469126efe on
          node-3                           94a836b0-6386-4010-9e78-ec1993192784 on


#### Try to fence on of the node

        # pcs stonith fence node-3
          Node: node-3 fenced

#### Stonith also can be ON/OFF using pcs property command

      # pcs property --all |grep stonith-enabled
        stonith-enabled: true


[root@node1 ~]# pcs stonith update xvmfence node1.linjb.com node2.linjb.com node3.linjb.com

#### Check output of pcs 

        # pcs status
          Cluster name: mycluster
          Stack: corosync
          Current DC: node1.linjb.com (version 1.1.16-12.el7-94ff4df) - partition with quorum
          Last updated: Mon Oct  8 19:23:33 2018
          Last change: Mon Oct  8 18:41:08 2018 by root via cibadmin on node1.linjb.com

          3 nodes configured
          7 resources configured

          Online: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

          Full list of resources:

           Clone Set: dlm-clone [dlm]
               Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
           Clone Set: clvmd-clone [clvmd]
               Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
           xvmfence	(stonith:fence_xvm):	Started node1.linjb.com

          Daemon Status:
            corosync: active/enabled
            pacemaker: active/enabled
            pcsd: active/enabled
