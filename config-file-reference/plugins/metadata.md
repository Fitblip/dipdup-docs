---
description: Plugins block
---

# metadata

This is an optional section used by the [metadata](https://github.com/dipdup-net/metadata) indexer plugin. It uses [`contracts`](../contracts.md) and [`datasources`](../datasources.md) aliases as well as the [`database`](../database.md) connection.

Metadata configuration has two required sections: `settings` and `indexers`

{% content-ref url="../../advanced/metadata-plugin.md" %}
[metadata-plugin.md](../../advanced/metadata-plugin.md)
{% endcontent-ref %}

## Settings

```yaml
metadata:
  settings:
    ipfs_gateways:
      - https://cloudflare-ipfs.com
    ipfs_timeout: 10
    http_timeout: 10
    max_retry_count_on_error: 3
    max_cpu: 4
    thumbnail:
      max_file_size_mb: 100
      workers: 20
      size: 100
  indexers:
    ...
```

#### ipfs\_gateways

An array of IPFS gateways. The indexer polls them sequentially until it gets a result or runs out of attempts. It is recommended to specify more than one gateway to overcome propagation issues, rate limits, and other problems.

#### ipfs\_timeout

How long DipDup will wait for a single IPFS gateway response. Default value is **10 seconds**.

#### http\_timeout

How long DipDup will wait for a HTTP server response. Default value is **10 seconds**.

#### max\_retry\_count\_on\_error

If DipDup fails to get a response from IPFS gateway or HTTP server, it will try again after some time, until it runs out of attempts. Default value is **3 attempts**.

**max\_cpu**

Maximum number of CPUs that can be executing simultaneously. Default value is **all availiable CPUs**.

**thumbnail.size**

Created thumbnails size in pixels. Size n×n  Default: 100.

**thumbnail.workers**

Count of thumbnail service parallel workers

**thumbnail.max\_file\_size\_mb**

Maximum file size which be downloaded from IPFS. Default: 50.

## Indexers

You can index several networks at once, or go with a single one. Indexer names are not standardized, but for clarity it's better to stick with some meaningful keys:

```yaml
metadata:
  settings:
    ...
  indexers:
    mainnet:
      filters:
        accounts:
          - contract_alias
      datasources:
        tzkt: tzkt_mainnet
```

Each indexer object has two keys: `filters` and `datasources` (required).

### Filters

#### accounts

Array of [contract](../contracts.md) aliases used to filter Big\_map updates by the owner contract address.

### Datasources

Metadata plugin is tightly coupled with [TzKT](../datasources.md#tzkt) provider.

#### tzkt

An alias pointing to a [datasource](../datasources.md) of kind `tzkt` is expected.
