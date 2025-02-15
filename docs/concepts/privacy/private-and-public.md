---
description: Public and private transactions
---

# Public and private transactions

GoQuorum achieves transaction privacy by:

- Enabling transaction senders to create private transactions by marking who is privy to a transaction via the
  `privateFor` parameter.
- Storing encrypted private data off-chain in a separate component called the
  [private transaction manager](index.md#private-transaction-manager).
  The private transaction manager encrypts private data, distributes the encrypted data to other parties that are privy
  to the transaction, and returns the decrypted payload to those parties.
- Replacing the payload of a private transaction with a key for the location of the encrypted payload, such that the
  original payload isn't visible to participants who aren't privy to the transaction.

!!! note

    While GoQuorum introduces the notion of "public transactions" and "private transactions," it does not introduce new
    transaction types.
    Rather, it extends the Ethereum transaction model to include an optional `privateFor` parameter (the inclusion of
    which results in GoQuorum treating a transaction as private) and the transaction type has a new `IsPrivate` method to
    identify private transactions.

## Public transactions

Public transactions have payloads that are visible to all participants of the same network.
These are
[created as standard Ethereum transactions](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethsendtransaction).

Examples of public transactions include market data updates from a service provider, or a reference data update such as
a correction to a bond security definition.

!!! Note

    GoQuorum public transactions are not transactions from the public Ethereum network.
    Perhaps a more appropriate term would be "common" or "global" transactions, but "public" is used to contrast with
    "private" transactions.

## Private transactions

Private transactions have payloads that are visible only to the network participants whose public keys are specified in
the `privateFor` parameter of the transaction.
`privateFor` can take multiple addresses in a comma-separated list.

When a GoQuorum node encounters a transaction with a non-null `privateFor` value, it sets the `v` value of the
transaction signature to `37` or `38` (as opposed to public transactions, whose `v` values are set according to
[EIP-155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)).

!!! note "Notes"

    - `privateFor` is not shared with other participants; it's only used to identify which nodes to send the encrypted payload to.
    - There's no direct restriction on private transaction size.
      As with public transactions, the only restriction is the gas limit.

See the [private transaction high-level lifecycle](private-transaction-lifecycle.md#normal-private-transactions).

You can enable private transactions by [configuring the private transaction manager connection](../../configure-and-manage/configure/private-transaction-manager.md),
and you can [send private transactions](../../tutorials/send-private-transaction.md).

## Public vs. private transaction handling

Public transactions are executed in the standard Ethereum way.
If a public transaction is sent to an account that holds contract code, each participant executes the same code, and
their state databases are updated accordingly.

Private transactions are executed differently: before the sender's GoQuorum node propagates the transaction to the rest
of the network, the node substitutes the original transaction payload with a key for the location of the encrypted
payload received from Tessera.
Participants privy to the transaction can replace the hash with the original payload via their Tessera instance, while
participants not privy only see the hash.

If a private transaction is sent to an account that holds contract code, participants not privy to the
transaction skip the transaction and don't execute the contract code.
Participants privy to the transaction replace the hash and call the virtual machine for execution, and their state
databases update accordingly.

!!! note

    See the [private transaction high-level lifecycle](private-transaction-lifecycle.md#normal-private-transactions) for an
    illustrated example.

As a result, these two sets of participants end up with different state databases and can't reach consensus.
To support this forking of contract state, GoQuorum stores the state of public contracts in a public state trie that
is globally synchronized, and the state of private contracts in a private state trie not globally synchronized.

### State verification

Block validation includes a check on the root of the public state trie to determine if the public state is synchronized
across nodes.
It also includes a check the global transaction hash, which is a hash of all public and private transactions in a block.
This means that each node can validate that it has the same set of transactions as other nodes.

Synchronization of the public state root and private transaction inputs (through the global transaction hash) can imply
synchronization of the private state across participating nodes.

To further validate that the private state change from a private transaction is the same across all participants, use
[`eth_storageRoot`](../../reference/api-methods.md#eth_storageroot) specifying the private smart contract address and
block height.
If the state is in sync across all participating nodes, they return the same root hash.

### Limitations

This model imposes a restriction on the ability to change public state in private transactions.
Since a common use case for a private contract is to read data from a public contract, the virtual machine changes to
read only mode for each call from a private contract to a public contract.
If the virtual machine is in read only mode, and the code tries to make a state change, the virtual machine stops
execution and throws an exception.
