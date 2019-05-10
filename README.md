# Celer dApp Contract

Celer dApps are highly interactive, secure and low-cost state-channel applications running on [Celer Network](www.celer.network) together with Celer [generic payment channel](https://github.com/celer-network/cChannel-eth). 

This repo provides templates and examples for developing the on-chain contract parts of dApps that can easily run on Celer mobile and web SDK [CelerX](https://celerx.app/). Note that most of the app interactiions happens off-chain, and these contracts are only used when players cannot reach consensus off-chain and want to dispute on-chain.

- **Multi-Session App:** initially deployed once by developer and can be repeatedly shared by all players til. No additional code needs to be deployed when players of on session wants to dispute on chain.

- **Single-Session App:** mostly used as an one-time virtual contract for fixed players without initial deployment. The player who wants to bring the off-chain game to on-chain dispute need to first deploy the contract.
 

## Latest Deployments

### Ropsten Testnet

#### MultiGomoku
- Contract address: [0xb352b23620ab8d75a05012aec0e0f5ce1015d743](https://ropsten.etherscan.io/address/0xb352b23620ab8d75a05012aec0e0f5ce1015d743)
- Deployed code: [MultiGomoku.sol](https://github.com/celer-network/cApps-eth/blob/3f471fd70a/contracts/gomoku/MultiGomoku.sol)
- Creator (owner): 0x674fa8ec8572f476f07b2bc7042e80a4f4d64107

## External API

#### API required by [CelerChannel](https://github.com/celer-network/cChannel-eth)

- [App with Boolean Result](https://github.com/celer-network/cApps/blob/master/contracts/templates/IBooleanResult.sol)

- [App with Numeric Result](https://github.com/celer-network/cApps/blob/master/contracts/templates/INumericResult.sol)

#### API required by [CelerX](https://celerx.app/).

- [Multi-session Apps](https://github.com/celer-network/cApps/blob/master/contracts/templates/IMultiSession.sol)


- [Single-session Apps](https://github.com/celer-network/cApps/blob/master/contracts/templates/ISingleSession.sol)


## Template Interface

We provide [templates](https://github.com/celer-network/cApps/tree/master/contracts/templates) to implement the common state-channel logics of external APIs, so that the developers can focus on the app-specific logic.

Developers using the provided templates **only need to implement the following interfaces**. For detailed usages, please refer to these [simplest example contracts](https://github.com/celer-network/cApps/tree/master/contracts/simple-app) and [tests](https://github.com/celer-network/cApps/tree/master/test/simple-app)

#### MultiSessionApp template interface

```javascript
/**
 * @notice Get the app result
 * @param _session Session ID
 * @param _query Query arg
 * @return True if query satisfied
 */
function getResult(bytes32 _session, bytes memory _query) internal view returns (bool) {}

/**
 * @notice Update on-chain state according to off-chain state proof
 * @param _session Session ID
 * @param _state Signed off-chain state
 */
function updateByState(bytes32 _session, bytes memory _state) internal returns (bool) {}

/**
 * @notice Update state according to an on-chain action
 * @param _session Session ID
 * @param _action Action data
 * @return True if update succeeds
 */
function updateByAction(bytes32 _session, bytes memory _action) internal returns (bool) {}

/**
 * @notice Finalize the session based on current state in case of on-chain action timeout
 * @param _session Session ID
 */
function finalizeOnTimeout(bytes32 _session) internal {}

/**
 * @notice Get app state associated with the given key
 */
function getState(bytes32 _session, uint _key) external view returns (bytes memory);
```

#### SingleSessionApp template interface

```javascript
/**
 * @notice Get the app result
 * @param _query Query args
 * @return True if query satisfied
 */
function getResult(bytes memory _query) public view returns (bool) {}

/**
 * @notice Update state according to an off-chain state proof
 * @param _state Signed off-chain app state
 * @return True if update succeeds
 */
function updateByState(bytes memory _state) internal returns (bool) {}

/**
 * @notice Update state according to an on-chain action
 * @param _action Action data
 * @return True if update succeeds
 */
function updateByAction(bytes memory _action) internal returns (bool) {}

/**
 * @notice Finalize based on current state in case of on-chain action timeout
 */
function finalizeOnTimeout() internal {}

/**
 * @notice Get app state associated with the given key
 */
function getState(uint _key) external view returns (bytes memory);
```

## Protobuf

We leverage Protocol Buffers to define a series of blockchain-neutral generalized data structures, which can be seamlessly used in off-chain communication protocols and instantly extended to other blockchains that we plan to support. We also developed and open sourced a Solidity library generator for decoding proto3 called [pb3-gen-sol](https://github.com/celer-network/pb3-gen-sol).

Below are the proto used by Celer dApps. [CelerX](https://celerx.app/) takes care of the protobuf encode and decode for app developers.

```protobuf
message AppState {
  // nonce should be unique for each app session among the same signers
  uint64 nonce = 1 [(soltype) = "uint"];
  // for each nonce, new state has higher sequence number
  uint64 seq_num = 2 [(soltype) = "uint"];
  // app specific state
  bytes state = 3;
}

message StateProof {
  // serialized AppState
  bytes app_state = 1;
  repeated bytes sigs = 2;
}

// used for multi-session app
message SessionQuery {
  // session ID
  bytes session = 1 [(soltype) = "bytes32"];
  // query related to the specified session
  bytes query = 2;
}
```