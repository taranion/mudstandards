---
sidebar_label: config
---
# The ``Config`` package

Source: [Aardwolf](https://www.aardwolf.com/wiki/index.php/Clients/GMCP#aard_config_options)

## config

A number of the game config commands can be queried and turned on or off via GMCP. The syntax, via GMCP, is:

### Requesting the current config setting 
````json
config [option name]
````

### Setting a configuration
````json
config [option name] <value>
````
- **option name**
  (*Mandatory*) 
- **value**
  (*Optional*) "on", "of" 
