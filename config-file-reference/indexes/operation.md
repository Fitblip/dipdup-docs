# operation

Operation index allows to query only those operations that are related to your DApp and to do pattern matching on the operation content \(internal calls chain\). This is a closest thing to fully-fledged event logs.

## Filters

DipDup supports filtering operations by `kind`, `source`_,_ `destination` __\(if applicable\), and `originated_contracts` \(if applicable\).

DipDup fetches only _applied_ operations.

#### contracts

```yaml
indexes:
  my_index:
    kind: operation
    datasource: tzkt
    contracts:
      - contract1
      - contract2
```

In this example DipDup will fetch all the operations where any of source and destination is equal to either _contract1_ or _contract2_ address. `contracts` field is obligatory, there has to be at least one contract alias \(from the [inventory](../contracts.md)\).

#### types

By default DipDup works only with transactions, but you can explicitly list operation types you want to subscribe to \(currently `transaction` and `origination` types are supported\):

```yaml
indexes:
  my_index:
    kind: operation
    datasource: tzkt
    contracts:
      - contract1
    types:
      - transaction
      - origination
```

Note, that in case of origination DipDup will query operations where either source or originated contract address is equal to _contract1._

## Handlers

Each operation handler contains two required fields:

* `callback` —  name of the _async_ function with a particular [**signature**](../../command-line/dipdup-init.md#handlers); DipDup will try to load it from the module with the same name `<package_name>.handlers.<callback>`
* `pattern` — a non-empty list of items that have to be matched

```yaml
indexes:
  my_index:
    kind: operation
    datasource: tzkt
    contracts:
      - contract1
    handlers:
      - callback: on_call
        pattern:
          - destination: contract1
            entrypoint: call        
```

You can think of operation pattern as a regular expression on a sequence of operations \(both external and internal\) with global flag enabled \(can be multiple matches\) and where various operation parameters \(type, source, destination, entrypoint, originated contract\) are used for matching.

### Pattern

Here are the supported filters for matching operations \(all optional\):

* `type` __— \(either _transaction_ or _origination_\) usually inferred from the existence of other fields
* `destination` — invoked contract alias \(from the [inventory](../contracts.md)\)
* `entrypoint` — invoked entrypoint name
* `source` — operation sender alias \(from the [inventory](../contracts.md)\)
* `originated_contract` — originated contract alias \(from the [inventory](../contracts.md)\)
* `similar_to` — originated contract has the same parameter and storage types as the reference one \(from the [inventory](../contracts.md)\)
* `strict` — stronger the `similar_to` filter by comparing the entire code rather than just parameter+storage
* `optional` — continue matching even if this item is not found \(with limitations, see below\)

**NOTE** that it's not required to match the entire operation content, you can skip external/internal calls that are not relevant. However, there is a limitation: optional items cannot be followed by operations ignored by the pattern.

```yaml
pattern:
  - destination: contract_1
    entrypoint: call_1
  - destination: contract_2
    entrypoint: internal_call_2
  - source: contract_1
    type: transaction
  - source: contract_2
    type: origination
    similar_to: contract_3
    strict: true
    optional: true
```

{% hint style="info" %}
Depending on whether you specify \(or not\) _destination+entrypoint_ for transactions and _originated\_contract_ for originations you will get slightly different callback argument types. Namely, in the first case DipDup will be able to generate the dataclass for a particular entrypoint/storage, and in the second case not \(meaning you will have to handle dynamically typed parameters/storage updates\).
{% endhint %}

