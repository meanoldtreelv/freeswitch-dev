# Using Dynamic or Static variables in the Dialplan

## Realtime (Inline) DB queries VS Database stored configurations
There are two different approaches to storing FreeSWITCH configuration in a database:

Mod_lua can be used to read the entire configuration from a JSON document or Postgres database row.
This is done when ReloadXML is called and builds a static XML file to be interpreted.

Inline (lua.so) is embedded within an XML document and is called when the dialplan segment is executed. This does not require a ReloadXML on information changes.  


## Managing the freqency of ReloadXML
Using dynamic variables in the dialplan allows for each call to be processed with the most up to date information without calling a ReloadXML and walking the entire dialplan. This is very useful for information that is very frequently updated (such as outbound caller ID [Strolid])

There are two issues with this approach being applied to the entire configuration;
  The first is that each varaible must be queried individually on every call. These queries must be made in realtime and prevent the call from proceeding while they are being executed.

  The second is that some modules store their own copy of a variable and is not refreshed unless a ReloadXML is called. An example of this would be mod_voicemail storing the VM password in its own reference table within the CoreDB.
