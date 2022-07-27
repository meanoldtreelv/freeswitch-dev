# Backends

#### There are several datatypes to be addressed when clustering FreeSWITCH

### Configuration
These are the XML/LUA files that make up the static configuration of the running FreeSWITCH server.
There are two main approaches to keeping the configurations in sync, and each approach has multiple implementations.
  - Filesystem Based
    - Use GlusterFS or CoroSync to replicate filesystem changes in realtime.
      - This is the easiest approach to implement.  
