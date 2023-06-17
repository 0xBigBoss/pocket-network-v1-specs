---
author: [0xbigboss](https://0xbigboss.github.com/)
version: 0.2.0-wip
---

# Pocket Network Governance Params Tech Spec

## Abstract

This document describes the proposed changes to the Pocket Network protocol to allow for the activation of governance parameters at specific block heights. A governance parameter in the Pocket Network protocol refers to a mechanism that enables or modifies specific features at a specific block height, without necessitating an upgrade to the protocol.

The following three types of features can be controlled using governance parameters:

- Governance controlled parameters: These parameters, modifiable by the DAO, control the behavior of the protocol.
- Alteration of protocol functionality: This refers to changes in the way the protocol functions, which can be enabled or modified.
- New consensus-breaking protocol versions: These are major changes to the protocol that can disrupt consensus if not managed properly.

The modification of a governance parameter is determined by the broadcasting of a `MessageGovernanceParamUpdate` by the DAO. The change takes effect immediately upon receiving majority approval from the governance committee.

## Governance Controlled Parameters

The Pocket Network protocol has a set of governance controlled parameters that can be modified by the DAO. These parameters control the behavior of the protocol, and their list can be found [TODO here](#TODO). For each parameter, there is an owner address elected by the DAO, and this address is the only one that can modify the parameter.

These parameters can be modified by broadcasting a `MessageGovernanceParamUpdate`. This message type is handled by the utility module, which is responsible for handling all governance messages.

Here is an example protobuf message for a `MessageGovernanceParamUpdate`:

```protobuf
message MessageGovernanceParamUpdate {
  string param_name = 1; // Name of the governance parameter

  oneof value {
    bool bool_value = 2;
    int64 int_value = 3;
    string string_value = 4;
    bytes bytes_value = 5;
  }
}
```

## Altering Protocol Functionality

Altering the functionality of the protocol using governance parameters follows a similar pattern to the governance controlled parameters. The key difference is that these changes can be more substantial, altering the fundamental operations of the network.

To modify the functionality of the protocol, a new governance parameter must be introduced. The activation or modification of these parameters must be proposed and voted upon by the governance committee.

## Consensus Breaking Protocol Versions

Consensus-breaking protocol versions introduce substantial changes to the protocol that may not be backward-compatible. These changes can disrupt the network consensus if not handled correctly.

Governance parameters offer a mechanism for introducing these changes in a controlled manner. The upgrade can be proposed and voted upon, allowing nodes to prepare and coordinate for the transition. Once the proposal is approved, nodes that have not upgraded will be unable to participate in the consensus process, ensuring the stability of the network.

To upgrade the protocol version, a `MessageProtocolUpgrade` must be broadcasted. This message requires the new version and the activation block height as inputs.

Here is an example protobuf message for a `MessageProtocolUpgrade`:

```protobuf
message MessageProtocolUpgrade {
  string new_version = 1; // New protocol version
  int64 activation_block_height = 2; // Block height for activation
}
```

## CLI and RPC

The CLI and RPC modules must be updated to handle the new message types related to governance parameters. They should provide functionalities to:

- Query the current state of a governance parameter.
- Propose a new governance parameter or an update to an existing one.
- Monitor the status of proposed governance parameters.

This ensures that network participants can smoothly interact with the new governance parameter system.

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
