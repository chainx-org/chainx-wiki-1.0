# ChainX Smart Contracts

## Run node

### 1. Accessing the test network

Please refer to the instructions of accessing the ChainX test network. After completing the relevant steps, the nodes should be synchronized to the latest and the corresponding configurations of the wallets completed.

### 2. Running local nodes

Please refer to the description of the ChainX Dev mode.

Having followed the steps, please ensure the number of blocks is over 150, because the first round rewards are issued to Alice after the 150th block.

If repeated tests are needed, please back up the data of blocks after the 150th. If you need a restart, please delete the old data and replace it with the backup data.

## Substrate Contracts, the Smart Contract Platform on ChainX

As the first technical framework in the blockchain domain, Substrate allows developers to focus on the logic of the chain instead of building the underlying infrastructure which is quite time-consuming. In addition, Substrate provides a number of functional modules by default, such as Staking and Consensus, which allows users to freely combine and customize these functions according to their needs. The contract module is one of the functional modules. A separate chain based on Substrate technology or a para-chain in the future can become a smart contract platform by integrating the contract module.

The ChainX smart contract platform is implemented by integrating Substrate's contract module. The main differences between the contract of ChainX and the default contract module of Substrate are as follows:

1. The contract-storing fees are cancelled. Contract-storing fees are charged simply after the contract is deployed onto the chain according to the size and the storage time of the contract. When certain account cannot afford the fees due to insufficient balance, the corresponding contract will be deleted and may not even be recovered. Even if the contract can be recovered, the success rate is extremely low, so it may cause great trouble to the contract development. Therefore, we decide to temporarily withdraw the contract-storing fees and only charge the Gas fee, which is consistent with Ethereum’s current charging mechanism. This storage fees could be redeployed after more improvements are made to the charging model.

2. Contracts are written by adapted chainx-org/ink. Chainx-org/ink fork compared with paritytech/ink, has a main change that is DefaultXrmlTypes replaces DefaultRrmlTypes to adapt ChainX’s runtime
* The association type Balance was changed from u128 to u64.
* Adapting the association type Call to support PCX transfers and converting XRC20 Tokens to ChainX inter-chain assets. The contract developers can design their own Dapps based on PCX and ChainX’s inter-chain assets.

Except the two changes, other contract writing methods are in line with paritytech/ink. Currently contracts are still written in ink1.0 which will be upgraded to ink2.0.

More resources on ink contract writing:

* Substrate-contracts-workshop: ink official tutorials
* Polkaworld-org/workshop: technical lectures on polkaworld contract

## Gas related parameters

```
Schedule {
    Version: 0, // configuration version
    Put_code_per_byte_cost: 200, // Gas required per byte when setting the wasm code
    Grow_mem_cost: 1, // Gas needed for memory increment in a single page
    Regular_op_cost: 1, // Gas needed by the ordinary operator Op
    Return_data_per_byte_cost: 1, // Gas consumed by return value per byte. The value should be carefully designed for the interoperability of contracts.
    Event_data_per_byte_cost: 20, // Gas consumed by event per byte in the contract
    Event_per_topic_cost: 1, // Gas consumed by each topic of event
    Event_base_cost: 1, // the base Gas consumed by each event in the contract, for example, Gas got reduced every time an event is executed.
    Call_base_cost: 60000, // the base Gas consumed by calling contracts, to put it in another word, Gas got reduced every time when a contract function is called.
    Instantiate_base_cost: 200000, // the base Gas consumed when a contract is initialized (instantiated)
    Sandbox_data_read_cost: 1, // Gas consumed by reading a byte in the contract
    Sandbox_data_write_cost: 1, // Gas consumed by writing a byte in the contract
    Max_event_topics: 4, // the maximum number of events in a single call
    Max_stack_height: 64 * 1024, // the maximum value of the stack in the contract
    Max_memory_pages: 16, // the maximum number of pages in the contract execution
    Max_table_size: 16 * 1024, // the maximum number of data structure tables in the contract
    Enable_println: false, // whether print is enabled in the contract
    Max_subject_len: 32, // maximum number of PRNG subjects
}
```

The default Gas Price in ChainX is 5

Please pay attention to `put_code_per_byte_cost`, `call_base_cost`, `instantiate_base_cost`. So in ChainX:

* Setting the WASM contract code is costly, so it’s highly recommended that contract developers should carefully design and modularize the contract code to avoid over-design.
* It takes lots of effects to instantiate a contract, so developers should try to reuse contracts that have been instantiated.
* Calling a contract also comes with costs, so developers are advised to streamline the contracts and focus on the logic.
* The maximum number of event is 4, so contract developers need to carefully design the event.

Deploying and calling the contract

For deploying and calling contracts, please refer to the relevant part of ChainX's wallet:

* Contract function
* Contract development and deployment components

## Developing and debugging contracts

We strongly recommend:

1. Contract developers debug contracts under the ChainX Dev node because many errors only appear in the node log.
2. Env.println is used in the contract in Dev mode, but env.println must be deleted in the main network and test network.
3. Contract developers use RPC to simulate contract execution first because contracts, whether executed or not, can be debugged via RPC without packaging.

### Some unexpected errors

Contracts-related log contains `[runtime|xrml_xcontracts]`, so if you only focus on the contract and the execution results, you only need to pay attention to grep:

```bash
tail -f log/chainx.log | egrep "xcontracts|apply_extrinsic"
```

If you encounter any problems in relation to the setting, deploying, and debugging of the contract, please refer to the following list for possible solutions:

#### 1. Contract code setting related issues

##### 1. Gas limit is not enough

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: there is not enough gas for storing the code
```

Generally `put_code_per_byte_cost` in ChainX is set relatively large, so the contract deployer should pay attention to whether the gas limit provided is sufficient. In general, it should be greater than `len(wasm) * put_code_per_byte_cost`

##### 2. Wasm contract, when called, is too large, causing it to be cut short, which leads to incomplete deployment.

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: Can't decode wasm code
```

At this point, please check if component calls cut wasm short.

##### 3. The wasm contract already exists on the chain. For the same contract, the code should not be set repeatedly.

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: the code is already stored on chain
```

When a developer writes, please check whether the contract already exists in the chain via sdk interfaces before uploading the code.

##### 4. The contract containing ext.print can be deployed in its own test, but cannot on the test network and the main network.

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: module imports `ext_println` but debug features disabled
```

In the test network and the main network, you must delete ext.print from the contract, or use conditional compilation control.

#### 2. Contract code deployment related issues

##### 1. When a contract instance already exists. Do not repeat the instantiation.

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: Alive contract or tombstone already exists
```

In the ChainX Contracts model, the way to generate a contract address:

```
Hash(code_hash + hash(input_data) + deployer)
```

So if a deployer uses same instantiation parameters on the same contract, the final result will be the same. Therefore if you need to use the same wasm code and parameters to instantiate another instance, please change your account.

##### 2. Wrong parameters; or parameters crash during instantiation, unable to proceed.

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: during execution|Failed to invoke an exported function for some reason|wrong selector, decode params fail or inner error
```

Please check the parameters and contract code, such as:

* The parameter that can be received by instantiation is u128, but the one being sent is u64.
* The storage in the contract when uninitialized goes under the operation of addition, subtraction, or overflow.

3. The deployment reached the gas limit

See the following errors

#### 3. Contract code call related issues

##### 1. Contract call or internal crash

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: during execution|Failed to invoke an exported function for some reason|wrong selector, decode params fail or inner error
```

1. The selector in calling contract does not match, or the parameter encoding is wrong.

 The selector must be called according to abi.json or old_abi.json generated by the compiled contract. If the selector does not exist, the call will be unsuccessful. The same holds true for coding errors.

2. Unexpected errors occur in the method of calling, such as using uninitialized storage

    ```rust
    Struct Incrementer {
    Value: storage::Value<u32>,
    }
    Impl Deploy for Incrementer {
        Fn deploy(&mut self) {
            // not init value
    }
    }
    Impl Incrementer {
       /// Flips the current state of our smart contract.
       Pub(external) fn inc(&mut self, by: u32) {
           // see this in the log
           Env.println(&format!("Incrementer::inc by = {:?}", by));
           Self.value += by; // call uninitialized storage
       }
    }
    ```

Therefore, debugging is required because the parameter/selector errors are internal errors of the contract. It’s easy to pinpoint the errors in the log.

3. Overflow or panic

Please refer to the code for details.

##### 2. Reaching gas limit during calling

```
[runtime|chainx_runtime::xexecutive|183L] [apply_extrinsic] failed: during execution|Failed to invoke an exported function for some reason|reach gas limit
```

Please try to reach an appropriate gas limit in the non-packaging process through the rpc interface chainx_contractCall. It is recommended to cover the farthest execution branch in the contract execution.

Through the grep log:

```
[runtime|xrml_xcontracts::gas] [refund_unused_gas]|account:88dc3417d5058ec4b4503e0c12ea1a0a89be200fe98922423d4334014fa6b0ee (5SjUJPJJ...)|gas_payment:2500000|refund:0|real cost:2500000|gas_spent:500000
```

You can see gas consumption in this call, where

1. gas_payment is the value of gas_limit*gas_price, which is the PCX temporarily stored by the caller.
2. refund is the PCX returned after the execution is completed.
3. real cost is the actual consumption of PCX
4. gas_spent is the consumption of gas

Note: in this part, consumption refers to the consumption of gas in the execution, and the actual payment of the PCX should plus the external handling fees for calling the contract.

## Inter-chain assets in ChainX’s smart contracts

Currently the ChainX smart contracts only support bitcoins.

The ChainX smart contracts adopt the “local currency + token” model to take multi tokens onboard, because of the similarities between Substrate Contracts’ smart contract platform and Ethereum’s.

In this model, ChainX's inter-chain assets on smart contracts adopt the contract token model, so PCX, like Ethereum provides gas and local currency circulation for smart contracts and inter-chain assets are introduced in the form of contract tokens.

Reasons for adopting token models instead of introducing tokens directly:

1. The Ethereum token contract is relatively mature, so it is easier for contract developers to migrate the Ethereum contract to the ChainX smart contract platform under the token model.
2. If the multi-token concept is introduced, Substrate Contracts will be greatly modified, which easily leads to unexpected troubles.

At present, ChainX’s smart contracts adopt the XRC20 model for bitcoins. According to contract developers’ feedback, other models may be adopted in the future.

1. XRC20 (formerly Ethereum ERC20)
2. XRC721 (formerly Ethereum ERC721)
3. XRC777 (formerly Ethereum ERC777)

The conversion between ChainX multi-token assets and tokens in contracts

## ChainX’s bitcoin X-BTC

Currently X-BTC corresponds to the token model XRC20 used in the smart contract platform. Users are free to convert ChainX's X-BTC to XRC20 in the contract platform and vice versa.

ChainX modified the ERC20 contract standard (XRC20) by adding two interfaces, issue and destroy. In addition to that, when a contract is deployed to the chain, it has to be instantiated with the contract instance uniquely specified, therefore, token instances other than the specified instance are not recognized by ChainX's Runtime.

* Issue can only be called by convert_to_xrc20 in Runtime, which cannot be called by calling the contract directly. After convert_to_xrc20 is called, the user's assets will be transferred to the contract instance, and the corresponding amount will be automatically issued to the account that sent the transaction.
* Destroy can be called by users to transfer the assets in the contract to the ChainX asset module. First, the corresponding tokens in the contract are destroyed, and then the contract will automatically call convert_to_assets to return the assets from the contract to the user, but convert_to_assets cannot be called through an external transaction.

### XRC20 on ChainX

XRC20 project

The XRC20 project deployed on ChainX is:

Https://github.com/chainx-org/xrc20

For XRC20, ChainX has provided 2 corresponding RPCs:

* chainx_contractXRCTokenInfo
* chainx_contractXRC20Call

#### ChainX wallet supports XRC20 abi

The current wallet has not yet fully integrated XRC20's abi, so developers need to add it manually.

1. Get the address of the specified XRC20-BTC contract instance on the chain:

Call rpc chainx_contractXRCTokenInfo to see the public key of XRC20-BTC contract address in the return value.

2. Refer to Section 3 of the [Contract Features Section | Contract Instantiation] (wallet#2. Contract Instantiation (Deploying Contracts) to fill in the public key of ERC20-BTC contract address via “adding an existing contract” and then upload the abi file target/abi.json.

#### Main network and test network

The main network and the test network should be deployed and set up beforehand according to the contract instance of X-BTC so that contract developers or users only need to input the abi file target/abi.json to the contract platform to invoke the contract. The address of the XRC20 can be obtained by rpc chainx_contractXRCTokenInfo.

#### ChainX Dev nodes

For the ChainX Dev nodes, the contract is not built within. So debugging and bitcoin related operations on smart contracts need to go a step further:

1. Set up the XRC20-BTC contract in ChainX Dev
2. Issue fake X-BTC for testing

A script has been provided for the two things. Search for README for more information.

The script is executed by default using Alice's private key (seeing the beginning of the ChainX Dev pattern). Please note that the script is not executed until after 150 blocks (it’s when Alice has enough assets to enable transactions).

The script also allows users to claim rewards from Alice certifiers if they don’t have sufficient amount of PCX.

#### Wallet function

ChainX’s new wallet provides the two-way conversion function for the inter-chain asset X-BTC and XRC20-BTC in the contract. Link for more information

## Deployment and Testing of ChainX’s Smart Contracts

Please refer to the ChainX Wallet for the deployment and calling of smart contracts.

Please refer to the ChainX Dev mode for testing.
