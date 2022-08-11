# Backends

#### There are several datatypes to be addressed when clustering FreeSWITCH

### Configuration
These are the XML/LUA files that make up the static configuration of the running FreeSWITCH server. Also included in this heading are the "static" recording files such as those used for announcements and IVRs.
There are two main approaches to keeping the configurations in sync, and each approach has multiple implementations.
  - Filesystem Based
    - Use GlusterFS or CoroSync to replicate filesystem changes in realtime.
      - This is the easiest approach to implement.
      - Using this method all changes are sent to all call routing nodes at the same time without further intervention
      - no change control, unable to undo changes without implementing temp files and additional filesystem logic
      - would need to use webhooks/pubsubhubub to notify calling nodes of changes, requries manual creation
      - can take advantage of "only load changes" feature of reloadxml
    - Use Git to manage filesystem changes
      - changes can be made on a filesystem not connected to a calling node.
      - changes can be made across multiple endpoints independent of eachother.
      - undo functionality via git pull
      - batch application of changes via git push
      - using repo webhooks/pubsubhubub calling nodes can be notified of pushes and automatically reload the configurations
      - can take advantage of "only load changes" feature of reloadxml
  - Database Backed
      - Using Postgres/relational db
        - LUA files on disk call the database when read to build the configuration.
        - schema changes would require rewriting LUA logic for all customers, updates would affect all customers.
        - easiest to query and update one specific configuration field
        - **can not** take advantage of "only load changes" feature of reloadxml
      - using MongoDB/NoSQL
        - LUA files on disk are significantly simpler as they retrieve the entire formatted document from Database
        - Schema changes are easy as LUA files on disk would not need to be changed.
        - **can not** take advantage of "only load changes" feature of reloadxml
    -Hybrid Backed
      - NoSQL+Bash+git
        - SSP/Admin changes can be written to database
        - Shell script is called on a "Save" action to update static XML files that will be read by calling nodes
        - Shell script also calls git commit and git push once XML files have been updated  
        - Calling nodes get updated via webhoot/pubsubhubub and daemon issues reloadxml
        - **can** take advantage of "only load changes" feature of reloadxml

### Realtime Info
Realtime data stored by FreeSWITCH includes sip registrations, agent statuses, active channels, queue positions, etc. This data needs to be accessable at all times to all FreeSWITCH calling nodes so that new call sessions can be routed and handled accordingly.
In a vanilla installation this is stored as a SQLite database on disk. There is support built into the core code to move this over to a Postgres server, either locally via unix socket or via remote connection. Individual Modules use their own database connections (defaulting to SQLite) and would need to be pointed to remote database during config.
This information is best queried from the FreeSWITCH Event Socket instead of from the database directly.
  - Clustered Postgres
    - Masterless (CockroachDB)
      - designed to be an N+1 scalable postgres solution
      - based on the work of 2ndquadrant Bi-directional replication
      - deployable on Kubernetes or Docker Swarm
      - Inteligent load sharing keeping the most frequently used data on the closest nodes while still maintaining n+1 copies of all data.
      - Proxies requests the shard does not have locally and stores the tables/rows requested for future lookups
      - No SPOF as all cluster nodes can be used as a read/write master
      - hardware failure does not result in downtime
      - recovery is automatic without intervention
      - expansion  of the cluster can be done online
    - Proxied (pgpool2)
      - 1+N replication (1 Master / N Slaves)
      - all nodes have a complete copy of the data and can be isolated
      - designed on Docker Swarm
      - Single Point of Failure in the stateless Proxy.
      - Hardware failure of the proxy results in 1-2 seconds of downtime while rebuild is in progress on new Hardware
      - Hardware failure of the Master node results in near instantanious failover, no queiries are missed.
      - Recovery from failed Master node requires manual intervention.
      - expansion of the cluster requires that the proxy be rebuilt resulting in downtime

### Voicemail Files
Voicemail files need to be synced across all calling servers. This is not quite as easy as the configuration files, as there is also a database that indexes all voicemail files to track their statuses.
mod_voicemail is set up for a SQLite db by default and has an option to use a ODBC DSN. Any database connection would need to support such DSN connenctions.
  - Database Backend
    - **Clustered Postgres (see above)**
      - most recent versions of mod_voicemail also support postgres nativly when specified in the DSN
    - Clustered MySQL solution using ndbd
      - requires further research
      - with latest updates to mod_voicemail may not be best Solutions
  - Filesystem Backend
    - GlusterFS
      - this is the goto solution based on other FreeSWITCH admins
      - silent instant replication of voicemail files across the cluster.
      - large storage requirement as there will be as many complete copies of the voicemail recordings as there are call servers
      - SSP can access files via mount on non call-server
    - **DB backed FUSE**
      - use a NoSQL database with a FUSE driver to store the voicemail files
      - Reccomendataions are either MongoDB or CouchDB
      - Significant storage array savings as 1+N copies of the data exist rather than 1 per calling node.
      - SSP can access the files via database call
    - Direct to DB
      - encode the DB files as base64 text and store in relational DB table
      - store as binary blob in NoSQL Database
      - requries re-implmentation of mod_voicemail with changes made by FusionPBX

### CDR Data
Unlike Asterisk, FreeSWITCH stores CEL data as part of the CDR rather than using a seperate database.
There are four different CDR modules avalible, three of which should be considered
  - mod_cdr_sqlite
    - This is the default module that writes cdr to a local sqlite database without the option for rotation. It is effectively useless for our purposes
  - mod_cdr_csv
    - This module writes CDR data to CSV files on disk.
    - Supports time based rotation without HUP
    - Would need to be ingested as the CDR data and muxed as the data would be call server specific
  - mod_cdr_pq_csv
    - this module writes CDR data to a postgres compatabile database
      - on failure it records to either CSV or SQL file on disk
    - calls are indexed by uuid generated from the call session data, so multiple servers should not overwite row entries by using the same index id.
    - Using CockroachDB the database size of the local instance on the callserver can be kept small while a much larger set of database node can store historic data.
        - This should be its own cluster instead of the one used for Realtime Data
    - most similar to the current setup with Asterisk
  - mod_cdr_mongodb
    - easiest to configured
    - writes each CDR entry as its own document within the Database
    - native MongoDB driver
    - Very Easy to scale database horozontally.
    - can use Unix Socket for increased performance
