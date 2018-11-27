### gfs2 cluster on rhel7


#### Package install on the all nodes

	# yum install -y pacemaker pcs psmisc policycoreutils-python

	# setenforce 1

	# cat /etc/selinux/config |grep SELINUX |grep -v "#"
	  SELINUX=enforcing
	  SELINUXTYPE=targeted

#### start pcsd services on the all nodes

	# systemctl start pcsd

	# systemctl enable pcsd
	  Created symlink from /etc/systemd/system/multi-user.target.wants/pcsd.service to /usr/lib/systemd/system/pcsd.service.

	# systemctl status pcsd
	  ● pcsd.service - PCS GUI and remote configuration interface
  	    Loaded: loaded (/usr/lib/systemd/system/pcsd.service; enabled; vendor preset: disabled)
   	    Active: active (running) since Sat 2018-10-06 17:31:59 IST; 59s ago
 	    Main PID: 9084 (pcsd)
   	    CGroup: /system.slice/pcsd.service
              └─9084 /usr/bin/ruby /usr/lib/pcsd/pcsd > /dev/null &

	    Oct 06 17:31:47 node2.linjb.com systemd[1]: Starting PCS GUI and remote configuration interface...
	    Oct 06 17:31:59 node2.linjb.com systemd[1]: Started PCS GUI and remote configuration interface.

#### Set hacluster password on the all nodes


	# passwd hacluster
	  Changing password for user hacluster.
	  New password: 
	  BAD PASSWORD: The password is shorter than 8 characters
	  Retype new password: 
	  passwd: all authentication tokens updated successfully.

#### Set Firewalld rules on the all nodes


	# firewall-cmd --add-service=high-availability --permanent
	  success

	# firewall-cmd --reload
	  success

#### Login to any of the cluster node and authenticate “hacluster” user.


	# pcs cluster auth node1.linjb.com node2.linjb.com node3.linjb.com
	  Username: hacluster
	  Password: 
	  node1.linjb.com: Authorized
	  node3.linjb.com: Authorized
	  node2.linjb.com: Authorized

#### Create new cluster run to any node

	# pcs cluster setup --name mycluster node1.linjb.com node2.linjb.com node3.linjb.com
	  Destroying cluster on nodes: node1.linjb.com, node2.linjb.com, node3.linjb.com...
	  node2.linjb.com: Stopping Cluster (pacemaker)...
	  node1.linjb.com: Stopping Cluster (pacemaker)...
	  node3.linjb.com: Stopping Cluster (pacemaker)...
	  node3.linjb.com: Successfully destroyed cluster
	  node2.linjb.com: Successfully destroyed cluster
	  node1.linjb.com: Successfully destroyed cluster

	  Sending 'pacemaker_remote authkey' to 'node1.linjb.com', 'node2.linjb.com', 'node3.linjb.com'
	  node3.linjb.com: successful distribution of the file 'pacemaker_remote authkey'
	  node1.linjb.com: successful distribution of the file 'pacemaker_remote authkey'
	  node2.linjb.com: successful distribution of the file 'pacemaker_remote authkey'
	  Sending cluster config files to the nodes...
	  node1.linjb.com: Succeeded
	  node2.linjb.com: Succeeded
	  node3.linjb.com: Succeeded

	  Synchronizing pcsd certificates on nodes node1.linjb.com, node2.linjb.com, node3.linjb.com...
	  node1.linjb.com: Success
	  node3.linjb.com: Success
	  node2.linjb.com: Success
	  Restarting pcsd on the nodes in order to reload the certificates...
	  node3.linjb.com: Success
	  node2.linjb.com: Success
	  node1.linjb.com: Success

#### Check pcs status

	# pcs status
	  Error: cluster is not currently running on this node

#### Start cluster on all nodes

	# pcs cluster start --all
	  node3.linjb.com: Starting Cluster...
	  node2.linjb.com: Starting Cluster...
	  node1.linjb.com: Starting Cluster...

#### Enable cluster on all nodes 

	# pcs cluster enable --all
	  node1.linjb.com: Cluster Enabled
	  node2.linjb.com: Cluster Enabled
	  node3.linjb.com: Cluster Enabled

#### Now check pcs status

	# pcs status
	  Cluster name: mycluster
	  WARNING: no stonith devices and stonith-enabled is not false
	  Stack: corosync
	  Current DC: node1.linjb.com (version 1.1.16-12.el7-94ff4df) - partition with quorum
	  Last updated: Sat Oct  6 18:31:06 2018
	  Last change: Sat Oct  6 18:28:27 2018 by hacluster via crmd on node1.linjb.com

	  3 nodes configured
	  0 resources configured

	  Online: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

	  No resources


	  Daemon Status:
	   corosync: active/enabled
	   pacemaker: active/enabled
	   pcsd: active/enabled

#### Verify Corosync configuration:


#### Check the corosync communication status

	# corosync-cfgtool -s
	  Printing ring status.
	  Local node ID 1
	  RING ID 0
		  id	        = 192.168.100.51
		  status	= ring 0 active with no faults

	  In my setup, first RING is using interface

	# ifconfig 
	  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
		inet 192.168.100.51  netmask 255.255.255.0  broadcast 192.168.100.255
		inet6 fe80::5054:ff:fe1b:4ec7  prefixlen 64  scopeid 0x20<link>
		ether 52:54:00:1b:4e:c7  txqueuelen 1000  (Ethernet)
		RX packets 12768  bytes 1411118 (1.3 MiB)
		RX errors 0  dropped 63  overruns 0  frame 0
		TX packets 12708  bytes 2435843 (2.3 MiB)
		TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

	  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
		inet 127.0.0.1  netmask 255.0.0.0
		inet6 ::1  prefixlen 128  scopeid 0x10<host>
		loop  txqueuelen 1  (Local Loopback)
		RX packets 1727  bytes 311764 (304.4 KiB)
		RX errors 0  dropped 0  overruns 0  frame 0
		TX packets 1727  bytes 311764 (304.4 KiB)
		TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

#### We can have multiple RINGS to provide the redundancy for the cluster communication. (We use to call LLT links in VCS )

#### Check the membership and quorum API’s.

	# corosync-cmapctl  | grep members
	  runtime.totem.pg.mrp.srp.members.1.config_version (u64) = 0
	  runtime.totem.pg.mrp.srp.members.1.ip (str) = r(0) ip(192.168.100.51) 
	  runtime.totem.pg.mrp.srp.members.1.join_count (u32) = 1
	  runtime.totem.pg.mrp.srp.members.1.status (str) = joined
	  runtime.totem.pg.mrp.srp.members.2.config_version (u64) = 0
	  runtime.totem.pg.mrp.srp.members.2.ip (str) = r(0) ip(192.168.100.52) 
	  runtime.totem.pg.mrp.srp.members.2.join_count (u32) = 1
	  runtime.totem.pg.mrp.srp.members.2.status (str) = joined
	  runtime.totem.pg.mrp.srp.members.3.config_version (u64) = 0
	  runtime.totem.pg.mrp.srp.members.3.ip (str) = r(0) ip(192.168.100.53) 
	  runtime.totem.pg.mrp.srp.members.3.join_count (u32) = 1
	  runtime.totem.pg.mrp.srp.members.3.status (str) = joined

#### Cluster should have quorum


	# corosync-quorumtool -s
	  Quorum information
	  ------------------
	  Date:             Sat Oct  6 18:41:37 2018
	  Quorum provider:  corosync_votequorum
	  Nodes:            3
	  Node ID:          1
	  Ring ID:          1/24
	  Quorate:          Yes

	  Votequorum information
	  ----------------------
	  Expected votes:   3
	  Highest expected: 3
	  Total votes:      3
	  Quorum:           2  
	  Flags:            Quorate 

	  Membership information
	  ----------------------
	  Nodeid      Votes Name
	       1          1 node1.linjb.com (local)
	       2          1 node2.linjb.com
	       3          1 node3.linjb.com


### iSCSI Client Installation and Configuration all of nodes on cluster

#### Install iscsi-initiator-utils package on all nodes

	# yum install  iscsi-initiator-utils

#### Change to the same IQN you set on the iSCSI target server

	# cat /etc/iscsi/initiatorname.iscsi 

	  InitiatorName=iqn.2017-18.iscsi.server:iscsi.linjb.com

#### iscover target on the nodes

	# iscsiadm -m discovery -t sendtargets -p 192.168.100.54
	  192.168.100.54:3260,1 iqn.2017-18.iscsi.server:tar1

#### Verify and confirm status after discovery

	# iscsiadm -m node -o show
	  BEGIN RECORD 6.2.0.874-2
	  node.name = iqn.2017-18.iscsi.server:tar1
	  node.tpgt = 1
	  node.startup = automatic
	  node.leading_login = No
	  iface.hwaddress = <empty>
	  iface.ipaddress = <empty>
	  iface.iscsi_ifacename = default
	  iface.net_ifacename = <empty>
	  ...
	  ...
	  ...
	  node.conn[0].iscsi.HeaderDigest = None
	  node.conn[0].iscsi.IFMarker = No
	  node.conn[0].iscsi.OFMarker = No
	  # END RECORD
 
#### login to the target on the nodes

	# iscsiadm -m node --login
	  Logging in to [iface: default, target: iqn.2017-18.iscsi.server:tar1, portal: 192.168.100.54,3260] (multiple)
	  Login to [iface: default, target: iqn.2017-18.iscsi.server:tar1, portal: 192.168.100.54,3260] successful.

#### Verify and confirm the established session

	# iscsiadm -m session -o show
	  tcp: [1] 192.168.100.54:3260,1 iqn.2017-18.iscsi.server:tar1 (non-flash)

#### confirm the partitions all the nodes

	# cat /proc/partitions 
	  major minor  #blocks  name

	  11        0    3963904 sr0
	   8        0   10485760 sda
	   8        1     524288 sda1
	   8        2    9960448 sda2
	 253        0       4096 dm-0
	 253        1    7958528 dm-1
	 253        2    7958528 dm-2
	 253        3    6938624 dm-3
	 253        4    1019904 dm-4
	 253        5    7958528 dm-5
	   8       16    5242880 sdb

#### Verify using lsblk all the nodes

	# lsblk 
	  NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	  sda                       8:0    0   10G  0 disk 
	  ├─sda1                    8:1    0  512M  0 part /boot
	  └─sda2                    8:2    0  9.5G  0 part 
	    ├─rhel-pool00_tmeta   253:0    0    4M  0 lvm  
	    │ └─rhel-pool00-tpool 253:2    0  7.6G  0 lvm  
	    │   ├─rhel-root       253:3    0  6.6G  0 lvm  /
	    │   ├─rhel-swap       253:4    0  996M  0 lvm  [SWAP]
	    │   └─rhel-pool00     253:5    0  7.6G  0 lvm  
	    └─rhel-pool00_tdata   253:1    0  7.6G  0 lvm  
	      └─rhel-pool00-tpool 253:2    0  7.6G  0 lvm  
	        ├─rhel-root       253:3    0  6.6G  0 lvm  /
	        ├─rhel-swap       253:4    0  996M  0 lvm  [SWAP]
	        └─rhel-pool00     253:5    0  7.6G  0 lvm  
	  sdb                      8:16    0    5G  0 disk 
	  sr0                      11:0    1  3.8G  0 rom

#### Install required packages on all Nodes

	# yum install fence-agents-all lvm2-cluster gfs2-utils

	# lvmconf --enable-cluster

	# reboot

#### Set the no-quorum-policy of the cluster to freeze so that that when quorum is lost, the remaining partition will do nothing until quorum is regained – GFS2 requires quorum to operate.

	# pcs property set no-quorum-policy=freeze

#### Set up a dlm resource. This is a required dependency for the clvmd service and the GFS2 file system. 

	# pcs resource create dlm ocf:pacemaker:controld op monitor interval=30s on-fail=fence clone interleave=true ordered=true

#### Set up clvmd as a cluster resource.

	# pcs resource create clvmd ocf:heartbeat:clvm op monitor interval=30s on-fail=fence clone interleave=true ordered=true

#### Set up clvmd and dlm dependency and start up order. The clvmd resource must start after the dlm resource and must run on the same node as the dlm resource. 

	# pcs constraint order start dlm-clone then clvmd-clone
	Adding dlm-clone clvmd-clone (kind: Mandatory) (Options: first-action=start then-action=start)

	# pcs constraint colocation add clvmd-clone with dlm-clone 
	clvmd-clone with dlm-clone (score:INFINITY) (id:colocation-clvmd-clone-dlm-clone-INFINITY)

#### Verify resources status

	# pcs status resources
	  Clone Set: dlm-clone [dlm]
	      Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
	  Clone Set: clvmd-clone [clvmd]
	      Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]


#### Create label
	
	# parted --script /dev/sdb "mklabel msdos"

#### Create partiton
	
	# parted --script /dev/sdb "mkpart primary 0% 100%" 

#### Create LVM objects from a single cluster node.

	# pvcreate /dev/sdb1

	# vgcreate vg_cluster /dev/sdb1

	# lvcreate -l100%FREE -n lv_cluster vg_cluster
  	  Logical volume "lv_cluster" created.
	  
#### Format the volume with a GFS2 file system. In this example, my_cluster is the cluster name. This example specifies -j 3 to indicate 3 journals because the number of journals you configure must equal the number of nodes in the cluster. 

	# mkfs.gfs2 -p lock_dlm -t mycluster:gfs2 -j 3 /dev/vg_cluster/lv_cluster
	  /dev/vg_cluster/lv_cluster is a symbolic link to /dev/dm-6
	  This will destroy any data on /dev/dm-6
	  Are you sure you want to proceed? [y/n] y
	  Discarding device contents (may take a while on large devices): Done
	  Adding journals: Done 
	  Building resource groups: Done   
	  Creating quota file: Done
	  Writing superblock and syncing: Done
	  Device:                    /dev/vg_cluster/lv_cluster
	  Block size:                4096
	  Device size:               4.97 GB (1302528 blocks)
	  Filesystem size:           4.97 GB (1302524 blocks)
	  Journals:                  3
	  Resource groups:           21
	  Locking protocol:          "lock_dlm"
	  Lock table:                "ha_cluster:gfs2"
	  UUID:                      36f0676b-c45a-4237-a3fa-bc8b32db172b

#### Create a mountpoint

	# mkdir /fico
	
#### Create a Filesystem resource, which configures Pacemaker to mount and manage the file system. This example creates a Filesystem resource named fs, and creates the /mnt/gfs2share on both nodes of the cluster. 

	# pcs resource create fs_gfs2 Filesystem device="/dev/vg_cluster/lv_cluster" directory="/fico" fstype="gfs2" 		 options="noatime,nodiratime" op monitor interval=10s on-fail=fence clone interleave=true

	  Assumed agent name 'ocf:heartbeat:Filesystem' (deduced from 'Filesystem')

#### Configure a dependency and a startup order for the GFS2 file system and the clvmd service. GFS2 must start after clvmd and must run on the same node as clvmd

	# pcs constraint order start clvmd-clone then fs_gfs2-clone 
	  
	  Adding clvmd-clone fs_gfs2-clone (kind: Mandatory) (Options: first-action=start then-action=start)

	# pcs constraint colocation add fs_gfs2-clone with clvmd-clone

#### Verify constraints

	# pcs constraint show 
	  Location Constraints:
	  Ordering Constraints:
	    start dlm-clone then start clvmd-clone (kind:Mandatory)
	    start clvmd-clone then start fs_gfs2-clone (kind:Mandatory)
	  Colocation Constraints:
	    clvmd-clone with dlm-clone (score:INFINITY)
	    fs_gfs2-clone with clvmd-clone (score:INFINITY)
	  Ticket Constraints:

#### Verify that the resources are running on all nodes.

	# pcs status 
	  Cluster name: mycluster
	  Stack: corosync
	  Current DC: node2.linjb.com (version 1.1.16-12.el7-94ff4df) - partition with quorum
	  Last updated: Tue Nov 27 16:12:38 2018
	  Last change: Mon Nov 12 17:10:04 2018 by root via cibadmin on node1.linjb.com

	  3 nodes configured
	  10 resources configured

	  Online: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

	  Full list of resources:

	   Clone Set: dlm-clone [dlm]
	       Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
	   Clone Set: clvmd-clone [clvmd]
	       Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]
	   xvmfence	(stonith:fence_xvm):	Started node2.linjb.com
	   Clone Set: fs_gfs2-clone [fs_gfs2]
	       Started: [ node1.linjb.com node2.linjb.com node3.linjb.com ]

	  Daemon Status:
	    corosync: active/disabled
	    pacemaker: active/disabled
	    pcsd: active/enabled

#### Verify on all the nodes using df command

	[root@node3 ~]# df -hT | grep gfs2
	/dev/mapper/vg_cluster-lv_cluster gfs2      5.0G  398M  4.6G   8% /fico

	[root@node2 ~]# df -hT | grep gfs2
	/dev/mapper/vg_cluster-lv_cluster gfs2      5.0G  398M  4.6G   8% /fico

	[root@node1 ~]# df -hT | grep gfs2
	/dev/mapper/vg_cluster-lv_cluster gfs2      5.0G  398M  4.6G   8% /fico

#### Verify using mount command

	[root@node1 ~]# mount | grep /fico
	/dev/mapper/vg_cluster-lv_cluster on /fico type gfs2 

	[root@node2 ~]# mount | grep /fico
	/dev/mapper/vg_cluster-lv_cluster on /fico type gfs2 

	[root@node3 ~]# mount | grep /fico
	/dev/mapper/vg_cluster-lv_cluster on /fico type gfs2 
