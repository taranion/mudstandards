---
sidebar_label: MPCP (Option 202)
description: MPCP is a draft secure authentication mechanism between proxies and MUD servers 
---
# MUD Proxy Control Protocol

:::warning
This is just a protocol draft - it is unknown if there really are or have been any implementations of this.
:::

**Source**: [Github](https://github.com/Twisol/mud-proxy-protocol/blob/master/proxy-auth.md)

*This draft attempts to define an easy-to-implement, secure authentication mechanism between proxies and MUD servers that would allow a MUD server to trust that the connection coming from the proxy is legitimately from the proxy, and not from a malicious hacker trying to get around multiplaying restrictions.*

*The basic idea is a new telnet suboption that sends a specially formatted HMAC (using SHA-1 for the hash function) along with the client information. The HMAC is generated using a secret key shared between the proxy and the server. The inspiration for this comes from Amazon AWS, and more specifically a distilled version of this auth style that I read at http://www.thebuzzmedia.com/designing-a-secure-rest-api-without-oauth-authentication/*
