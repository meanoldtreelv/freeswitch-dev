XML files written to disk for add of extension or module to the dialplan
core config information is written to XML file as static value
edititable values are stored in RDB and queried during operation, not at build time.

GIT is not going to be used for versioning

Users are able to snapshot thier current config and save it in SMB
24 hour and 7 day rolling snapshots are also maintained

XML reload is only needed for add/remove extension or Dialplan flow changes.

Config changes can be queried directly from MongoDB?
Experementation is needed






CDR:
Add tag to all CDRs based off of initial Call-ID header to be able to query all child CDR entries
