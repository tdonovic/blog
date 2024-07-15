+++
author = "tdonovic"
date = 2020-06-01
title = "vault child tokens"
+++

To continue from last time: we need to create some vault policy to allow
terraform to create child tokens. Thanks to (tyrannosauraus-becks
)[https://github.com/tyrannosaurus-becks], who suggests to add this
policy to the github users that we have previously created:
{{< highlight go >}}
  path "auth/token/create" {
    capabilities = ["create", "read", "update", "delete", "list"]
  }
{{< / highlight >}}
Huh, look at that, it worked.
It's nice when things work the first time! Now to rid our repos of
secrets!
