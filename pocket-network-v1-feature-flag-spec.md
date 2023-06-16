---
author: [0xbigboss](https://0xbigboss.github.com/)
version: 0.1.0-wip
---

# Abstract

This document describes the proposed changes to the Pocket Network protocol to allow for the activation of feature flags. A feature flag is a way to enable or disable a feature in a software application. Feature flags are used to enable or disable features during runtime without deploying new code. This allows for the network to be more flexible and adapt to the needs of the community.

Feature flags as they relate to the Pocket Network protocol are a way to enable or disable features in the protocol without having to upgrade the protocol. This allows for the network to be more flexible and adapt to the needs of the community.

A feature with respect to Pocket Network can be one of the following:

- Governance controlled parameters
- Altering protocol functionality
- A new consensus breaking protocol version

Each feature above needs a block height to be activated. The block height is used to determine when the feature will be activated. A feature is enabled if the current block height is greater than or equal to the activation block height. A feature is disabled if the current block height is less than the activation block height. A feature can also be disabled by setting the activation block height to -1.

## Governance Controlled Parameters

The Pocket Network protocol has a set of governance controlled parameters that can be modified by the DAO. These parameters are used to control the behavior of the protocol. A list of the current governance controlled parameters can be found [TODO here](#TODO). For each parameter, there is an owner address elected to by the DAO. The owner address is the only address that can modify the parameter.

These parameters are currently controlled by the DAO and can be modified by submitting a governance transaction. The governance parameter change transaction is a message type that is handled by the utility module. The utility module is responsible for handling all governance transactions.

## Motivation

The following is separated into two sections, the first section describes the motivation for the activation of feature flags and the second section describes the motivation for the modification of governance parameters.

## Modules Affected

- Persistence
- Utility
  - New Message Type: `MessageFeatureFlagUpdate`
- CLI
- RPC

### Feature Flags

### Persistence

The persistence module currently has feature flags as part of [it's interface](https://github.com/pokt-network/pocket/blob/a29b654b808b0239d4ed3dc37131317168b8aaf0/shared/modules/persistence_module.go#L219-L221).

```go
// Flags
	GetIntFlag(paramName string, height int64) (int, bool, error)
	GetStringFlag(paramName string, height int64) (string, bool, error)
	GetBytesFlag(paramName string, height int64) ([]byte, bool, error)
```

There should be a simple boolean type flag included in this interface to allow for simple feature flags to be activated or deactivated.

### Utility

To introduce a new feature flag, a new feature flag name and activation block height must be included in a new Transaction Message type. The feature flag name is used to identify the feature flag and the activation block height is used to determine when the feature flag will be activated.

Feature flags can be activated or deactivated by submitting a governance transaction of type, `MessageFeatureFlagUpdate`.

An example protobuf message for a `MessageFeatureFlagUpdate` transaction is as follows:

```protobuf
message MessageFeatureFlagUpdate {
  string feature_flag_name = 1;
  int64 activation_block_height = 2;
}

```

A feature is enabled if the current block height is greater than or equal to the activation block height. A feature is disabled if the current block height is less than the activation block height. A feature can also be disabled by setting the activation block height to -1.

### Handling MessageFeatureFlagUpdate

The `MessageFeatureFlagUpdate` message type needs to be handled in the utility module.

https://github.com/pokt-network/pocket/blob/12f9f498c8b05a91f4c08c6b27227d94013d9232/utility/unit_of_work/tx_message_handler.go

A new param owner needs to be introduced to the utility module to allow for the activation of feature flags. The param owner is used to determine if the message sender is allowed to activate or deactivate feature flags.

https://github.com/pokt-network/pocket/blob/10a931a06f81902518a919ce33b9642f8388949c/utility/unit_of_work/tx_message_signers.go#L25-L26 https://github.com/pokt-network/pocket/blob/dbc0deb6a7aec39359745e9be952a4992689052a/build/config/genesis.json#L4242

# References

## RC 0.10.X Rehearsal

The [TestNet Rehearsal](https://www.notion.so/8c37c541c49243a186c95ff38655da68?pvs=21) has multiple references to feature flags and gov params that cross the boundary of protocol upgrades

### Enabling new gov param

```bash
HEIGHT=$(curl -s -X POST ${POCKET_ENDPOINT}/v1/query/height | jq '.height')
pocket gov enable $DAO $((HEIGHT+3)) BLOCK $NETWORK 10000 --remoteCLIURL ${POCKET_ENDPOINT}
```

### Changing enabled gov param

```bash
POKT gov change_param $DAO $NETWORK pocketcore/BlockByteSize "\"8000000\"" 10000 --remoteCLIURL ${POCKET_ENDPOINT}
```

### Upgrading protocol version

```bash
export HEIGHT=$(curl -s -X POST ${POCKET_ENDPOINT}/v1/query/height | jq '.height')
POKT gov upgrade $DAO $((HEIGHT+3)) 0.10.0 $NETWORK 10000
```

## Pocket-core source code

A good starting point with the v0 source code (and other) is:

1. Pocket-core: https://github.com/pokt-network/pocket-core/blob/b515c1d342d326d7f637f8a4b63e13f244112d33/codec/codec.go#L43
2. Cosmos: https://docs.cosmos.network/main/modules/upgrade

### Priority

See https://github.com/pokt-network/pocket/pull/766#discussion_r1204204100

![Screenshot 2023-06-13 at 12.00.30 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5cb4d96-2bfc-4d08-bdea-1b582a27f4b0/Screenshot_2023-06-13_at_12.00.30_PM.png)
