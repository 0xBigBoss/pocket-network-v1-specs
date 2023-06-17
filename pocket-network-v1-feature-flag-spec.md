---
author: [0xbigboss](https://0xbigboss.github.com/)
version: 0.1.0-wip
---

# Pocket Network Feature Flags Tech Spec

## Abstract

This document describes the proposed changes to the Pocket Network protocol to allow for the activation of feature flags at specific block heights. A feature flag in the Pocket Network protocol refers to a mechanism that enables or disables specific features at a specific block height, without necessitating an upgrade to the protocol.

The following three types of features can be controlled using feature flags:

- Governance controlled parameters: These parameters, modifiable by the DAO, control the behavior of the protocol.
- Alteration of protocol functionality: This refers to changes in the way the protocol functions, which can be enabled or disabled.
- New consensus-breaking protocol versions: These are major changes to the protocol that can disrupt consensus if not managed properly.

The activation of a feature is determined by the block height, with the feature being enabled if the current block height is greater than or equal to the activation block height. Conversely, the feature is disabled if the current block height is less than the activation block height. A feature can also be permanently disabled by setting the activation block height to -1.

## Governance Controlled Parameters

The Pocket Network protocol has a set of governance controlled parameters that can be modified by the DAO. These parameters control the behavior of the protocol, and their list can be found [TODO here](#TODO). For each parameter, there is an owner address elected by the DAO, and this address is the only one that can modify the parameter.

These parameters are controlled by the DAO and can be modified by submitting a governance transaction. The governance parameter change transaction is a message type that is handled by the utility module, which is responsible for handling all governance transactions.

To enable a new governance parameter, the `pocket gov enable` command can be used, which requires the block height (retrieved using the `curl -s -X POST ${POCKET_ENDPOINT}/v1/query/height` command) and the DAO as inputs. Once a parameter is enabled, it can be modified using the `POKT gov change_param` command.

## Feature Flags

### Persistence

The persistence module currently has feature flags as part of [it's interface](https://github.com/pokt-network/pocket/blob/a29b654b808b0239d4ed3dc37131317168b8aaf0/shared/modules/persistence_module.go#L219-L221). These flags can be of type int, string, or byte. To allow for simple feature flags to be activated or deactivated, a new boolean type flag should be added to this interface.

### Utility

Introducing a new feature flag involves including a new feature flag name and activation block height in a new Transaction Message type. The feature flag name identifies the feature flag, while the activation block height determines when the feature flag will be activated.

Feature flags can be activated or deactivated by submitting a governance transaction of type `MessageFeatureFlagUpdate`. This type of transaction includes the feature flag name and the activation block height as parameters.

A feature is enabled if the current block height is greater than or equal to the activation block height. A feature is disabled if the current block height is less than the activation block height. A feature can also be disabled by setting the activation block height to -1.

Here is an example protobuf message for a `MessageFeatureFlagUpdate` transaction:

```protobuf
message MessageFeatureFlagUpdate {
  string feature_flag_name = 1;
  int64 activation_block_height = 2;
}
```

### Handling MessageFeatureFlagUpdate

The utility module will handle the `MessageFeatureFlagUpdate` message type. A new param owner needs to be introduced to the utility module to allow for the activation of feature flags. The param owner determines if the message sender has the authority to activate or deactivate feature flags

## Altering Protocol Functionality

Altering the functionality of the protocol using feature flags will follow a similar pattern to the governance controlled parameters. The key difference is that these changes can be more substantial, altering the fundamental operations of the network.

To modify the functionality of the protocol, a new feature flag must be introduced with a corresponding block height at which the new functionality will take effect. The activation or deactivation of these feature flags must be proposed and voted upon in a similar fashion to the governance controlled parameters.

## Consensus Breaking Protocol Versions

Consensus-breaking protocol versions introduce substantial changes to the protocol that may not be backward-compatible. These changes can disrupt the network consensus if not handled correctly.

Feature flags offer a mechanism for introducing these changes in a controlled manner. The upgrade can be tied to a specific block height, allowing nodes to prepare and coordinate for the transition. Once the block height is reached, nodes that have not upgraded will be unable to participate in the consensus process, ensuring the stability of the network.

To upgrade the protocol version, the `POKT gov upgrade` command can be used, which requires the block height (retrieved using the `curl -s -X POST ${POCKET_ENDPOINT}/v1/query/height` command), the DAO, and the new version as inputs.

## CLI and RPC

The CLI and RPC modules must be updated to handle the new transaction message types related to feature flags. They should provide functionalities to:

- Query the current state of a feature flag.
- Propose a new feature flag or an update to an existing one.
- Submit votes for proposed feature flags.
- Monitor the status of proposed feature flags and their votes.

This ensures that network participants can smoothly interact with the new feature flag system.

The detailed design and specification of these CLI and RPC updates will be covered in future versions of this spec.

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
