## Bitcoin Bridge

ChainX's Bitcoin Bridge uses a one-way light node relay mode. User funds are co-hosted by multiple trust nodes. There are two hot and cold signing addresses, both of which are public addresses.

## Bitcoin Light Node

ChainX obtains the longest chain based on Bitcoin's POW consensus algorithm, and obtains valid deposit/withdrawal transactions based on the transaction root in the block. The valid address for deposit and withdrawal monitoring is the latest hot multi-signature address.

#### Default allocation

* Block time: 600 seconds
* Number of transaction confirmation blocks: 4
* Starting monitoring block height: 576576
* Withdrawal fee: 0.001 BTC
* Maximum withdrawals: 100
* Multi-sign ratio: equal to 2/3
* Cross-chain remark carrier: OP_RETURN

#### Transaction interface: submit block header btc_bridge/push_header

* Parameters: version, previous block hash, transaction root, timestamp, nonce, remarks
* Judgment: Anyone can call, the difficulty of the block must meet the requirements, the previous block hash already exists in the light node, and it does not accept forked blocks greater than the number of transaction confirmation blocks.

If it is a legal block, the longest chain is updated, and the transactions in the block that has reached the confirmed state are executed, including deposit confirmation, coin issuance, and withdrawal confirmation and destruction.

#### Transaction interface: submit transaction btc_bridge/push_tx

* Parameters: block hash, original transaction data, transaction root certificate, pre-order transaction original data, remarks
* Judgment: anyone can call, the block needs to exist, the transaction needs to be in the transaction root of the block, and the pre-order transaction needs to be the input of this transaction

If the transaction is valid and the block has not been confirmed, it is saved, and if the transaction is valid and the block has been confirmed, then it is executed.

#### Transaction interface: construct multi-signature withdrawal btc_bridge/xxxxx

* Parameters: withdrawal ID list, original text to be signed
* Judgment: It can only be called by the trust node. It is required that all withdrawals are in the withdrawal list, and the output of the original text to be signed corresponds to the user's withdrawal address and amount (deducting the handling fee).

Legally store, and wait for confirmation from other trust nodes

#### Transaction interface: respond to multiple signing withdrawals btc_bridge/xxxxx

* Parameters: new signature result, response (agree|reject)
* Judgment: It can only be called by the trust node. The new signature must be legal and there is no duplication, or it must be rejected.

If the new signature result reaches the multi-signature ratio, the withdrawal structure is completed, waiting for the relay program to broadcast to the Bitcoin network. If the multi-signature ratio is rejected, and the multi-signature withdrawal is deleted, the trust node needs to re-initiate.

## Bitcoin Relay

Anyone can call ChainX's submission block header and submit transaction interface. As long as it meets the needs of light nodes, it will be accepted by ChainX, so the community can have multiple versions of the relay program to prevent a single program from missing certain transactions.

ChainX will regularly count the list of nodes or users participating in the relay, and use it to announce to the community the names of nodes that maintain network security, increase community credit for them, and make up for their transaction fee losses.
