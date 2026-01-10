---
name: CDN Uploading
description: How to upload static assets to a CDN
---

When uploading static assets to a CDN, always:

1. If there is no bucket called `sherpa-sh-{github repo id}` create one.
2. In that bucket, create a new folder from a concatentation of the "github repo id" and the "github action run id".
3. Upload files to this folder, maintaining the structure of the folder being uploaded.

For example, if the github repo id is 123 and the github action run is 6967. The static assets folder should be
uploaded to `123-6967` in the bucket `sherpa-sh-123`.

