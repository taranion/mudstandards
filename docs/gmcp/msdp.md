---
sidebar_label: MSDP
---
# MSDP over GMCP

This package has been defined by Mudlet.

Source: [Mudhalla](https://tintin.mudhalla.net/protocols/gmcp/)

MSDP provides the client with a list of supported variables when requested, so it's not a requirement for MUD servers to provide documentation on the packages it supports.

MSDP over GMCP allows the client to choose between using either MSDP, GMCP, or both. In the case a client enables both MSDP and GMCP the client is expected to be able to process both MSDP and GMCP data interchangably, similarly the server is expected to process both MSDP and GMCP data interchangably, as is the case in MTH 1.5.

Keep in mind that JSON does not allow sending control-codes while MSDP does. This is automatically handled in MTH 1.5.

As things currently stand MSDP over GMCP is primarily used for client to client communication by TinTin++, offering scripters a more readable alternative to MSDP, with a minimal implementation and maintenance burden. MSDP over GMCP might also be useful for inter-mud standards.

:::warning
When using MSDP over GMCP the package name is considered case sensitive and MSDP must be fully capitalized. There are no subpackages as these are not necessary.
:::

Link: [The MSDP Protocol Definition](../mud/msdp)