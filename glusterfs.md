## GlusterFS Notes

### Volume types

- Distributed Volume
  - analog of Raid 0 storage array, one copy of each file in cluster, node failure results in loss of files

- Replicated Volume
  - analog of Raid 1 storage array, one copy of each file is kept on each node. Very space intensive, writes take longer the larger the "brick" count. loss of a node results in no data loss or performance penalty

- Distributed Replicated Volume
  - analog of Raid 0+1 storage array, creates a replicated copy of a distrubuted volume. built from a mirrord set of spanned volumes. losing one node will comprimise the entire dataset until a rebuild happens, offset with additional replication sets.

- Dispersed volume
  - closest analog would be a Raid 5 storage array, however instead of pairity data additonal rebuild chunks are stored so regardless of node failure a full intact copy of the file exists without reconstruction. offers the benifets of a Distributed Replicated Volume with significant storage space savings as the cluster grows in size. A Dispersed Volume cannot be grown in size.

  - Distributed Dispersed Volume
    - a spanned array of Dispersed Volumes similar to a raid 05 array. Created by expanding an existing Dispersed Volume. Must be created by adding an additional number of bricks equal to the count used in the first Dispersed Volume.
    - ex: Dispersed Volume was made of 5 bricks (replica 3) any additional Dispersed Volumes that are added to the Distributed group must also be 5 bricks in size with a replica value of 3.
    - note that expansion only cares about the brick count, not the brick size. If the initial bricks were 150GB in size, and the expansion bricks were 50Gb in size, the total avalible space in the example would be ~330GB (minus metadata overhead)
    - when expanding or shrinking the volume a file rebalance operation is reccomended, this can be done while the filesystem is online and is completely transparent.
    - Red Hat suggests a best practice of building Distributed Dispersed volume sets from a number of bricks that are all the same size based on expansion needs. for example if future expansions are expected to be in 50GB incriments, create the initial volume size using a number of 50GB bricks to create the initial load even if they are on the same server.


### Volume conclusions
we should be using a Distributed Dispersed Volume for voicemail storage. Individual Call servers should **NOT** be part of the storage array as losing a brick from the underlying subvolume would take the entire subvolume offline potentially having major impacts on cluster performance.

Calling nodes should however join the GlusterFS cluster allowing transparent connection to volumes using the native FUSE interface. These nodes joining/leaving do not have an impact on the cluster.

Node Arrangement:
  - Total number of nodes are
