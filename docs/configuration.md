# Cardano SL configuration

This document describes Cardano SL configuration. Configuration is
stored in YAML format, conventional name for the file is
`configuration.yaml`. This file contains multiple configurations, each
one is associated with a key.  An example of configuration can be
found in file `lib/configuration.yaml`.


## Prerequisites

* Read about [VSS certificates](https://cardanodocs.com/technical/pvss/#vss-certificate).
* You need to know the difference between
  [balance and stake](https://cardanodocs.com/cardano/balance-and-stake/).
* You need to know about AVVM. I don't know if it's described
  somewhere, so briefly: there was vending process after which we have
  a map from AVVM address (32 bytes, base64url encoded) to an integer
  number (amount of ADA). AVVM addresses can be converted to
  Cardano-SL addresses and initially each address has this amount of
  ADA (as balance, not stake).
* [Bootstrap era and stake locking](https://cardanodocs.com/timeline/bootstrap/).
* SSC is Shared Seed Computation, it's performed by rich nodes (with
  big stake) to agree upon a seed which will be used for leaders
  selection for an epoch.

## Genesis

In Cardano-SL we need to specify which addresses initially have ADA
and how much. It's traditionally called _genesis block_. In Cardano-SL
we also have various parameters of the algorithm along with initial
balances. The hash of this data (in canonical JSON format) is used as
the parent of the very first real block.

### Naming conventions

Traditionally all initial data is called _genesis block_. However, in
Cardano-SL _genesis block_ has a different meaning: it's the first
block in an epoch, as described in Ouroboros paper. There are many
genesis blocks (one per epoch).

For this reason we call the set of all initial balances, stakes and
parameters _genesis data_ (and there is `GenesisData` type in Haskell
code). Another term is _genesis spec_ (data type in Haskell is called
`GenesisSpec`) which specifies how to create _genesis data_. See below
for more details. When we say simply _genesis_ it usually means
_genesis data_, the actual meaning is usually clear from the context.

Note: we are not trying to force everyone to change standards and say
_genesis data_ instead of _genesis block_ (though it kinda makes
sense, because in Cardano-SL _genesis data_ is not a block, it
consists of a bunch of different data). It's ok to use term _genesis
block_ and usually the meaning will be clear from the context. But for
historical reasons developers prefer to use term _genesis data_ to
refer to the initial set of parameters and balances and _genesis
block_ to refer to the first block in an epoch. It's enforced by names
in code. It may be changed later.

### How to specify genesis

Configuration is used to specify genesis data used by the node. It is
part of `core` configuration (it's accessible by key
`<configuration-key>.core.genesis`). There are two ways to specify
genesis data.

* One way is to provide genesis data itself, stored in JSON
  format. Genesis data explicitly states balances of all addresses,
  all bootstrap stakeholders, heavyweight delegation certificates,
  etc. It makes it clearer which genesis is used, but it doesn't
  describe _how_ this genesis was constructed. Also it requires more
  steps to create genesis data, than to create a spec.  This mechanism
  is used for mainnet (both production and staging) so that everyone
  can open this file and see all balances (and everything else from
  genesis data) at any time. It also will be used for testnet, at
  least for the first one.  The format is demonstrated below. `hash`
  is the hash (Blake2b 256 bits) of canonically encoded
  `mainnet-genesis.json` file. It can be computed manually using
  `scripts/js/genesis-hash.js` script.

```
    genesis:
      src:
        file: mainnet-genesis.json
        hash: 5f20df933584822601f9e3f8c024eb5eb252fe8cefb24d1317dc3d432e940ebb
```

  The format of `mainnet-genesis.json` is described
  in [Genesis data format](#genesis-data-format).

* Another way is to provide a specification how to generate genesis
  data. It allows one to specify how many nodes should have stake,
  which bootstrap stakeholders should be used, what should be total
  balance, etc. When node is launched with spec as genesis, it
  automatically constructs genesis data from it. This way to specify
  genesis is supposed to be used for testing clusters. It's
  questionable whether it should be used for testnet and currently we
  can't use it for testnet for performance reasons. We don't use it
  for mainnet (and we won't have other mainnets).
  See [Genesis spec format](#genesis-spec-format) for details.

### Genesis data format

Let's start with an example:

```
{
    "avvmDistr": {
        "0cAZzmtB2CUjlsMULb30YcBrocvK7VhQjUk--MyY2_Q=": "8333333000000",
        ...
        "zE56EfVAvv0XekVyBDGh2Iz1QT6X38YxlcBYO20hqa0=": "1773135000000"
    },
    "blockVersionData": {
        "heavyDelThd": "300000000000",
        "maxBlockSize": "2000000",
        "maxHeaderSize": "2000000",
        "maxProposalSize": "700",
        "maxTxSize": "4096",
        "mpcThd": "20000000000000",
        "scriptVersion": 0,
        "slotDuration": "20000",
        "softforkRule": {
            "initThd": "900000000000000",
            "minThd": "600000000000000",
            "thdDecrement": "50000000000000"
        },
        "txFeePolicy": {
            "multiplier": "43946000000",
            "summand": "155381000000000"
        },
        "unlockStakeEpoch": "18446744073709551615",
        "updateImplicit": "10000",
        "updateProposalThd": "100000000000000",
        "updateVoteThd": "1000000000000"
    },
    "bootStakeholders": {
        "0d916567f96b6a65d204966e6aab5fbd242e56c321833f8ba5d607da": 1,
        "4bd1884f5ce2231be8623ecf5778a9112e26514205b39ff53529e735": 1,
        "5f53e01e1366aeda8811c2a630f0e037077a7b651093d2bdc4ef7200": 1,
        "ada3ab5c69b945c33c15ca110b444aa58906bf01fcfe55d8818d9c49": 1
    },
    "ftsSeed": "736b6f766f726f64612047677572646120626f726f64612070726f766f646120",
    "nonAvvmBalances": {},
    "protocolConsts": {
        "k": 2160,
        "protocolMagic": 60987900,
        "vssMaxTTL": 6,
        "vssMinTTL": 2
    },
    "heavyDelegation": {
        "c6c7fb227037a1719b9d871ea49b6039325aeb293915da7db9620e3f": {
            "cert": "0f2ff9d1f071793dbd90fce8d47e96a09b594ee6dd00a4bac9664e4bd6af89830035ec921698b774c779eb1b6a2772d3d6ae37e630c06c75fbfecd02a6410a02",
            "delegatePk": "I07+5HIUW0Lkq0poMMzGziuILwxSyTJ4lMSsoQz4HZunD5Xsx3MfBZWc4l+206lUOFaU+spdJg7MkmFKBoVV0g==",
            "issuerPk": "AZzlT+pC5M4xG+GuRHJEXS+isikmVBorTqOyjRDGUQ+Lst9fn1zQn5OGKSXK29G2dn7R7JCugfcUebr0Dq7wPw==",
            "omega": 1
        }
    },
    "startTime": 1505621332,
    "vssCerts": {
        "0d916567f96b6a65d204966e6aab5fbd242e56c321833f8ba5d607da": {
            "expiryEpoch": 1,
            "signature": "396fe505f4287f832fd26c1eba1843a49f3d23b64d664fb3c8a2f25c8de73ce6f2f4cf37ec7fa0fee7750d1d6c55e1b07e1018ce0c6443bacdb01fb8e15f1a0f",
            "signingKey": "ohsV3RtEFD1jeOzKwNulmMRhBG2RLdFxPbcSGbkmJ+xd/2cOALSDahPlydFRjd15sH0PkPE/zTvP4iN8wJr/hA==",
            "vssKey": "WCECtpe8B/5XPefEhgg7X5veUIYH/RRcvXbz6w7MIJBwWYU="
        },
        ...
    }
}
```

Section `"avvmDistr"` contains AVVM addresses with corresponding coins
values (lovelaces).

Section `"blockVersionData"` contains fundamental blockchain-related values:

*  `"heavyDelThd"` - heavyweight delegation threshold,
*  `"maxBlockSize"` - maximum size of block, in bytes,
*  `"maxHeaderSize"` - maximum size of block's header, in bytes,
*  `"maxProposalSize"` - maximum size of Cardano SL update proposal, in bytes,
*  `"maxTxSize"` - maximum size of transaction, in bytes,
*  `"mpcThd"` - threshold for participation in SSC (shared seed computation),
*  `"scriptVersion"` - script version,
*  `"slotDuration"` - slot duration, in microseconds,
*  `"softforkRule"` - rules for softfork:
   *  `"initThd"` - initial threshold, right after proposal is confirmed,
   *  `"minThd"` - minimal threshold (i.e. threshold can't become less than this one),
   *  `"thdDecrement"` - theshold will be decreased by this value after each epoch,
*  `"txFeePolicy"` - transaction fee policy's values,
*  `"unlockStakeEpoch"` - unlock stake epoch after which bootstrap era ends,
*  `"updateImplicit"` - update implicit period, in slots,
*  `"updateProposalThd"` - threshold for Cardano SL update proposal,
*  `"updateVoteThd"` - threshold for voting for Cardano SL update proposal.

Please note that values of all thresholds are multiplied by `10⁻¹⁵`. So if particular threshold is
defined as `X`, its actual value is `X * 10⁻¹⁵`.

Section `"bootStakeholders"` contains bootstrap era stakeholders'
identifiers (`StakeholderId`s) with corresponding weights.

Field `"ftsSeed"` contains seed value required for Follow-the-Satoshi
mechanism (hex-encoded).

Section `"protocolConsts"` contains basic protocol constants:

*  `"k"` - security parameter from the paper,
*  `"protocolMagic"` - protocol magic value (it's included into a
   serialized block and header and it's part of signed data, so when protocol
   magic is changed, all signatures become invalid) used to
   distinguish different networks,
*  `"vssMaxTTL"` - VSS certificates maximum timeout to live (number of epochs),
*  `"vssMinTTL"` - VSS certificates minimum timeout to live (number of epochs).

Section `"heavyDelegation"` contains an information about heavyweight delegation:

*  `"cert"` - delegation certificate,
*  `"delegatePk"` - delegate's public key,
*  `"issuerPk"` - stakeholder's (issuer's) public key,
*  `"omega"` - index of epoch the block PSK is announced in, it is
   needed for replay attack prevention, can be set to arbitrary value
   in genesis.

Keys in `heavyDelegation` dictionary are `StakeholderId`s of
certificates' issuers. They must be consistent with `issuerPk` values.

Field `"startTime"` is the timestamp of the 0-th slot.  Ideally it
should be few seconds later than the cluster actually starts. If it's
significantly later, nodes won't be doing anything for a while. If
it's slightly before the actual starts, some slots will be missed, but
it shouldn't be critical as long as less than `k` slots are missed.

Section "vssCerts" contains VSS certificates:

*  `"expiryEpoch"` - index of epoch until which (inclusive) the
   certificate is valid;
*  `"signature"` - signature of certificate,
*  `"signingKey"` - key used for signing,
*  `"vssKey"` - VSS public key.

Keys in `vssCerts` dictionary are `StakeholderId`s of
certificates' issuers. They must be consistent with `signingKey` values.

### Genesis spec format

Let's again start with an example:

```
    genesis:
      spec:
        initializer:
          testnetInitializer:
            testBalance:
              poors:        12
              richmen:      4
              richmenShare: 0.99
              useHDAddresses: True
              totalBalance: 600000000000000000
            fakeAvvmBalance:
              count: 10
              oneBalance: 100000
            distribution:
              testnetRichmenStakeDistr: []
            seed: 0
        blockVersionData:
          scriptVersion:     0
          slotDuration:      7000
          maxBlockSize:      2000000
          maxHeaderSize:     2000000
          maxTxSize:         4096 # 4 Kb
          maxProposalSize:   700 # 700 bytes
          mpcThd:            0.01 # 1% of stake
          heavyDelThd:       0.005 # 0.5% of stake
          updateVoteThd:     0.001 # 0.1% of total stake
          updateProposalThd: 0.1 # 10% of total stake
          updateImplicit:    10 # slots
          softforkRule:
            initThd:        0.9 # 90% of total stake
            minThd:         0.6 # 60% of total stake
            thdDecrement:   0.05 # 5% of total stake
          txFeePolicy:
            txSizeLinear:
              a: 155381 # absolute minimal fees per transaction
              b: 43.946 # additional minimal fees per byte of transaction size
          unlockStakeEpoch: 18446744073709551615 # last epoch (maxBound @Word64)
        protocolConstants:
          k: 2
          protocolMagic: 55550001
          vssMinTTL: 2
          vssMaxTTL: 6
        ftsSeed: "c2tvdm9yb2RhIEdndXJkYSBib3JvZGEgcHJvdm9kYSA="
        heavyDelegation: {}
        avvmDistr: {}
```

Most values are the same as in _genesis data_. It's easier to write about
differences:
1. Genesis spec has `initializer` value not present in genesis
   data. See
   [Balances, stakes, initializer](#balances-stakes-initializer) for details.
2. Thesholds are specified as floating-point numbers.
3. `ftsSeed` is base64-encoded, not hex-encoded.
4. `txFeePolicy` is specified slightly differently. The comments in
   the example explain the format pretty well. Note that the values
   are lovelaces, not ADA.
5. TODO: what about `heavyDelegation`?
6. TODO: what about `avvmDistr`?

### Balances, stakes, initializer

In _genesis data_ there are two fields which determine balances:
`avvmDistr` and `nonAvvmBalances`. The former is a map from AVVM
addresses to balances, the latter is a map from Cardano-SL addresses
to balances.

Stakes are computed from balances based on `bootStakeholders`
map. Keys in these map are `StakeholderId`s which will have stake and
values are their weights (stake will be distributed proportionally to
those weights).

In _genesis spec_ there are two fields which determine balances and
stakes: `avvmDistr` and `initializer`.

`avvmDistr` is supposed to contain the result of vending (which can be
found in `lib/configuration.yaml`, see `mainnet_avvmDistr`), but it
also can be empty if you don't want need AVVM addresses.

`initializer` can be either `testnetInitializer` or
`mainnetInitializer`. There are several differences between these two
(**TODO** it's not finalized).

* `testnetInitializer` includes `testBalance` field which allows to
  add entries to `nonAvvmBalances` and specify how many richmen should
  exist.
* **TODO** add more!
* Also `mainnetInitializer` includes `startTime`, while for `testnetInitializer`
  it should be passed from command line. It's not related to balances
  and stakes, but should be mentioned as well.

**TODO** The rest of this section is WIP, please skip
to [System start time](#system-start-time).

The key point of `testnetInitializer` is that it contains seed which
is used to generate secret keys. It's quite convenient for testnet,
because there we want everyone to be able to have some coins. So we
can deterministically generate a lot of addresses with some coins and
let people use them. It's also convenient for devnet clusters
(i. e. clusters primarily used by developers to test something),
because in this case we also want to know all secrets. An example of
`testnetInitializer` was provided above. The most interesting part
there is `distribution` field:

```
  distribution:
   testnetRichmenStakeDistr: []
```

It means that generated richmen will also be bootstrap stakeholders
and will be participating in SSC. Note that richmen's keys are
deterministically generated, so basically everyone has full control
over system in this case. It's suitable for devnet where we don't care
about adversaries, but it's not suitable for testnet, because we don't
want to give full control over the system to arbitrary people.

For testnet there is another possible value of `distribution`:
`testnetCustomStakeDistr`. It allows one to explicitly specify
bootstrap stakeholders and VSS certificates. Example:

```
  distribution:
    testnetCustomStakeDistr:
      bootStakeholders:
        75fc2050ea497eb615461d01170679d143fb70c07c0baf3db54c6237: 1
        eaba957b871c4d5d9f5eab1a4183a037b3b0a59e31a52185d37627f1: 1
      vssCerts:
        75fc2050ea497eb615461d01170679d143fb70c07c0baf3db54c6237:
          expiryEpoch: 5
          signature: "34c3f15a59997a3b95d022ec223999ade1824e0ec709e2fcef6d7e92c614f9f8297cc377b71e54825226d84792867a0067000f20a7600c1eb2b368e6b8cc4602"
          signingKey: "4vHA0HXrmpD6VhhCe44CAJ8IHr9BKpCUtOmGRZu/39vRLJ1z2vJp8h+rdOqb7tJg7Uzf94x0NlM8xHgmhPBecg=="
          vssKey: "WCECml0Nc9bjjerxHGf2sDfKJIILyDKvjM8zCdfzBmYxcyU="
        eaba957b871c4d5d9f5eab1a4183a037b3b0a59e31a52185d37627f1:
          expiryEpoch: 5
          signature: "628a5076967851da90bedb36f5c8cc38d1a3fe66f09acdfa9be02c6ed7910480156e1a7d0d8fe67f2604caa404146798807c35ee0cde4111ced742d7f53b590a"
          signingKey: "FXU35aHyAKouHgj+qWi/vfC8CzU4JI9+zxDkQwq+B4sFIsaai3+U307SFLp2zxZB7Jm9kjc19TcBtTefkHxxmg=="
          vssKey: "WCEDNB6FR+wZABJptA2JiReaIHLBRVz04Ys3P6UMDuSW8Qg="
```

`mainnetInitializer` doesn't have any seed, in this case all data is
specified explicitly, not generated somehow. It contains bootstrap
stakeholders, vss certificates, non-avvm balances and system start
time (see below). Example:

```
    initializer:
        mainnetInitializer:
        startTime: 1505621332000000
        bootStakeholders:
            0d916567f96b6a65d204966e6aab5fbd242e56c321833f8ba5d607da: 1
        vssCerts:
            0d916567f96b6a65d204966e6aab5fbd242e56c321833f8ba5d607da:
                expiryEpoch: 1
                signature: "396fe505f4287f832fd26c1eba1843a49f3d23b64d664fb3c8a2f25c8de73ce6f2f4cf37ec7fa0fee7750d1d6c55e1b07e1018ce0c6443bacdb01fb8e15f1a0f"
                signingKey: "ohsV3RtEFD1jeOzKwNulmMRhBG2RLdFxPbcSGbkmJ+xd/2cOALSDahPlydFRjd15sH0PkPE/zTvP4iN8wJr/hA=="
                vssKey: "WCECtpe8B/5XPefEhgg7X5veUIYH/RRcvXbz6w7MIJBwWYU="
            4bd1884f5ce2231be8623ecf5778a9112e26514205b39ff53529e735:
                expiryEpoch: 2
                signature: "773dcdf727d05720a76e7f88ef1c8a629399a30a98943eac761f9706a0e45def608765362202ce571b9394167c310a445a84745695d73e89086254a4c5be610c"
                signingKey: "oUDzVUqwmXH4E3RlDrS4zhZ6kwv9rnNiwe8dI6lIg794+bWQBlULwvnwiIGgK4z0HT8+o+nru8F5xDy4ZL2/lA=="
                vssKey: "WCEChSQx6z4OxYrHNYbu5GGztX4FBBxGMfzmO6C+xNTrDVw="
        nonAvvmBalances: {}
```

Such type of genesis spec was used once to construct mainnet genesis
data. It's not clear whether it will be ever used again.

### System start time

System start time is taken from configuration or command line option
(`--system-start`). It's taken from configuration if genesis is
provided as genesis data (which always contains system start time) or
if genesis is provided as genesis spec with `mainnetInitializer`. The
reason why system start is part of `mainnetInitializer`, but not part
of `testnetInitializer` is that mainnet is launched rarely (actually
only once, but there is also staging), but other clusters are launched
more often, and it's easier to change this value from CLI.

### Tools

There are some tools relevant to genesis data.

* `cardano-keygen` has a command to dump all secret keys and similar
  data to files. Usage: `cardano-keygen --system-start 0
  --configuration-file <file> --configuration-key <key>
  generate-keys-by-spec --genesis-out-dir <dir>`. This command will
  generate and dump secrets to `<dir>`. To deploy a cluster you need keys
  of core nodes. Just use `generate-keys-by-spec`
  to obtain these keys if you have a spec with `testnetInitializer`.

* `cardano-node-simple` and `cardano-node` have command line option to
  dump genesis data (in JSON format). The option is
  `--dump-genesis-data-to genesis.json` (data will be dumped to
  `genesis.json`). It can be used to generate _genesis data_ from
  _genesis spec_ if you want it to be used by nodes or just want to
  verify generated data.

### Generating genesis for testnet

**TODO**: likely outdated.

There is `testnet_public_full` configuration which is almost suitable
for testnet. The only values to be changed are inside
`testnetCustomStakeDistr`: `bootStakeholders` and `vssCerts`. They
depend on secret keys of core nodes which shouldn't be publicly
known. You need to generate secret keys and VSS certificates and put
them into configuration. There is also `testnet_staging_full` which
has different `k`, `protocolMagic` and some other values.

To generate a secret key use `cardano-keygen --system-start 0
generate-key --path <SECRET_KEY_PATH>`. Generate as many keys as there
should be core nodes. Last line looks like this:

> [keygen:INFO] Successfully generated primary key and dumped to secrets/testnet/node4.sk, stakeholder id: 39f2bb9fd75ac348e6c92467e61eb3e1418a394f5a2105f9e014b666, PK (base64): /fbQqCqUPdPImYA2djkGYMpj9HZGkDeKhTc/mLcPG7sdhJSR6Ou0sB3J5OII2VdWTXSHRM1cPdNwZsaNba4rcg==

You should use stakeholder ids as keys in `bootStakeholders`
map. Values are stakeholders' weights, having all weights equal to 1
is fine.

The second step is VSS certificates generation. Use `cardano-keygen
--system-start 0 --configuration-key testnet_public_full generate-vss
--path <SECRET_KEY_PATH>` to generate VSS certificate. This how it
looks like:

> JSON: key 75fc2050ea497eb615461d01170679d143fb70c07c0baf3db54c6237, value {"expiryEpoch":5,"signature":"b735325b8f21a033cbe3005c35e4397dd33168c62c05cc8f59e66efbcdeb36b5862d7a324def1ba10e4a79cf0e56c75568c84e6ca28b1a9bb5da65fc6a8cb002","signingKey":"4vHA0HXrmpD6VhhCe44CAJ8IHr9BKpCUtOmGRZu/39vRLJ1z2vJp8h+rdOqb7tJg7Uzf94x0NlM8xHgmhPBecg==","vssKey":"WCECml0Nc9bjjerxHGf2sDfKJIILyDKvjM8zCdfzBmYxcyU="}

**IMPORTANT**: you must pass correct `--configuration-key`, because
certificate validity depends on `protocolMagic`. So if you are
generating certificates for staging, use `testnet_staging_full`
instead.

Put this data into `vssCerts` map. There should be as many VSS
certificates as there are core nodes (i. e. do it for each secret).

### Generating genesis for mainnet

**TODO**: we have some scripts and `README.md` in
`scripts/prepare-genesis`. We probably will never need them anymore,
but we need to preserve them to be able to review at any point for
example. Probably that `README.md` should be moved to this section or
to this folder (`docs/`). Some part of that `README.md` has been moved
to this file already.

## Core configuration (besides genesis)

**TODO**

## Node configuration

Node configuration is accessible by `<configuration-key>.node` key. It
has the following values:

* `networkDiameter` (seconds) — how much time it takes to propagate a
  block across the network. In practice this value determines when
  slot leaders create a block (it's done `networkDiameter` seconds
  before the end of the slot).
* `mdNoBlocksSlotThreshold` — number of slots after which node will
  actively requests blocks if it doesn't receive any blocks without
  requesting them (in normal cases blocks are announced). Should be
  less than `2 · k`.
* `recoveryHeadersMessage` — how many headers will be sent in a
  batch. This value should be greater than `k`.

**TODO**: describe the rest.

## Command line options

Almost all executables accept several options to specify
configuration.
* `--configuration-file` is used to specify path to configuration
file.
* `--configuration-key` specifies key in this configuration.
* `--system-start` specifies system start time as unix timestamp in
  seconds. It must be provided if it's not available from
  configuration and must not be provided otherwise.
* `--configuration-seed` can be passed to override `seed` value from
  `testnetInitializer`. In `testnetInitializer` isn't used, passing
  `--configuration-seed` is prohibited.

## Our configurations

### `mainnet` configurations

There are several mainnet configurations in `lib/configuration.yaml`
file for different keys.

* `mainnet_base` configuration serves as a basis for other mainnet
  configurations and shouldn't be used directly. It also can be used
  as a basis for non-mainnet configurations.
* `mainnet_example_generated` configuration is an example of mainnet
  configuration where genesis is provided by spec. It shouldn't be used
  directly, only as an example.
* `mainnet_full` configuration is what core nodes use in real
  mainnet. `mainnet_wallet_win64` and `mainnet_wallet_macos64` are
  almost same, but they have different `update` configurations
  (different application name, version, etc.).  They should be used by
  wallets (i. e. nodes launched with Daedalus).
* `mainnet_dryrun_full` is like `mainnet_full`, but for
  staging. `mainnet_dryrun_wallet_win64` and
  `mainnet_dryrun_wallet_macos64` are for staging wallets.
* `mainnet_staging_short_epoch_full` is very similar to
  `mainnet_dryrun_full`, but it has much smaller `k` which implies
  much shorter epoch.

### `internal_staging` configurations

`internal_staging_full` configuration can be used by full nodes in
internal staging. It's

`internal_staging_wallet_win64` and `internal_staging_wallet_macos64`
are very similar configurations for wallets.

**Obsolete**

There are two configurations for testnets. They are ready to be used,
except `bootStakeholders` and `vssCerts`. Testnet genesis preparation
is described above. Bootstrap stakeholders and vss certificates in
`lib/configuration.yaml` are derived from secret keys from
`secrets/testnet/`. Since they are publicly avaiable, they shouldn't
be used for real testnet. Note that `vssCerts` differ even though
secret keys are same. That's because certificates validity depends on
protocol magic.

**Sketch**

Workflow for testnet: modify `seed`, dump keys, figure out system
start, dump genesis data, use it.

### `devnet` configuration

There is a configuration called `devnet` which should be used to setup
clusters for developers or QA to test something. There are some values
which should be set carefully before using this configuration, they
depend on particular task:

* `slotDuration`, `k` are parameters of the protocol, set to
  reasonable defaults, but sometimes we may want to use different
  values. Reminder: number of slots in an epoch is `10 · k`.
* `mdNoBlocksSlotThreshold` and
  `recoveryHeadersMessage` depend on `k`. Set their values properly if
  you modify `k`. See [node configuration](#node-configuration).
* `genesis.spec.initializer.testnetInitializer.richmen` is basically
  the number of core nodes. Should be the same as the number of core
  nodes deployed.

### Other configurations

There are few more configurations which should be briefly
mentioned. `dev` configuration is used by developers to launch cluster
locally. It's very simple and provides bare minimum. `bench`
configuration is used for benchmarks. `test` configuration is used in
tests by default (it's embedded into binary). `default` configuration
is an alias for `dev`.
