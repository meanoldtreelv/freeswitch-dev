# Backends

#### There are several datatypes to be addressed when clustering FreeSWITCH

### Configuration
These are the XML/LUA files that make up the static configuration of the running FreeSWITCH server.
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
