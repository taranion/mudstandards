# MUDStandards agent instructions

This is a static website using the Docusaurus framework, with the Mermaid graph plugin

## Setup commands
- Install dependencies: `npm install; npm install mermaid; npm install --save @docusaurus/theme-mermaid`
- Test building locally: `npm run build`
- Start local server: `npm run serve`

## Repository layout
- Directory "docs" contains full text descriptions of existing standards of the MUD community.
  Subdirectories classify the type of standard - e.g. Telnet options are in "telnet", GMCP packages in "gmcp"
- Directory "howto" contains short tutorials
- "src" directory has some Docusaurus specific config and with "custom.css" the sites CSS file
- "static" directory keeps static files like images referenced from multiple places.