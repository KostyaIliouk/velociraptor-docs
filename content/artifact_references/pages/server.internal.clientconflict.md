---
title: Server.Internal.ClientConflict
hidden: true
tags: [Internal Artifact]
---

This event artifact is an internal event stream receiving events
about client conflict.

When two clients attempt to connect to the server with the same
client id, the server rejects one of these with a 409 Conflict HTTP
message. The client id will be forwarded on this artifact as well so
the server may take action.


```yaml
name: Server.Internal.ClientConflict
description: |
  This event artifact is an internal event stream receiving events
  about client conflict.

  When two clients attempt to connect to the server with the same
  client id, the server rejects one of these with a 409 Conflict HTTP
  message. The client id will be forwarded on this artifact as well so
  the server may take action.

type: INTERNAL

```
