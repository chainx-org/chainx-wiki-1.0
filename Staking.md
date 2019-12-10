
## Starting Configurations

- Validator node 
   - 4 at minimum and 100 at maximum.
   - Validator nodes are responsible for packaging transactions and maintaining consensus on the chain.

- Trustee node (currently only Bitcoin Trustee)
  - 3 at least and 15 at most. ( Bitcoin Trustee)
  - The concept of a trustee node is based on the inter-chain transfer bridge. Trustee nodes often come at hand for hosting users' inter-chain assets when ChainX light client cannot be integrated. Trustee nodes are elected by ChainX's validators in trustee elections.

- Dividend cycle 
  - 150 blocks, about 5 minutes.
  - A certain number of PCXs are issued per dividend cycle.

- Election cycle
  - mainnet: 12 dividend cycles, 1,800 blocks, about 60 minutes

- Lock-up period for vote retraction 
  - Lock-up period for retracted votes: 72 election cycles, 3 days on the mainnet
  - Lock-up period for redeemed nodes: 720 election cycles, 30 days on the mainnet
  - When a user retracts a vote from a node, PCXs involved are locked up for a period before return to the balance. After the period expires, the user can manually unlock the amount.


## Basic concept

### Dividend cycle rewards

A dividend cycle is analogous to one block of Bitcoin. A certain amount of PCXs are to be issued as rewards per cycle.
Similar to Bitcoin, the total amount of PCX is 21 million, and the reward is halved every 210,000 dividend cycles, to put simply, 50 PCXs for the first 210,000 dividend cycles or the initial cycles, and 25 for the second round.

### Node rewards distribution in dividend cycles 

**Team rewards**

In the first 210,000 cycles, 20% of the 50 PCXs will be distributed to the team, which means the total rewards for all nodes in each cycle is 50 * 80% = 40 PCX.
The reward cap to the team is 10% of the total, therefore, the team will only enjoy rewards in the initial cycles, and none afterwards.

**Asset mining**

Except for the 20% of the rewards that are handed out as dividends in the first 210,000 cycles, the rest are distributed through asset mining.

All participants in asset mining compete against each other in terms of their computing power with PCX as computing unit. Mining consists of two kinds: virtual computing power of inter-chain mining and real computing power of voting mining:

- Inter-chain mining refers to all kinds of assets outside the chain like BTC and ETH that users transfer into the system through depositing, mapping or locking-up, that are automatically converted into the virtual PCX inter-chain mining power according to the price of the asset and the discount given to each asset. Every inter-chain asset has a different discount rate which could be adjusted through community voting. The reason why assets are discounted in value entering the chain is because PCX, as the system's internal currency, should enjoy greater mining power than other inter-chain assets to encourage users to hold more PCX.
- Voting mining means users that hold real PCX participate in the election of the PoS system and manually vote for certain nodes.

The total mining power of a user is equal to the inter-chain mining power of virtual PCX plus the voting power of the real PCX. For example, one user deposits one BTC, two ETHs into ChainX, and holds 700 PCXs among which 300 PCXs participate in the election and 400 PCXs do not. At the time, the exchange price in ChainX stands at 1BTC: 10000PCX, 1ETH: 1000PCX, and the user will automatically obtain the mining power of (1 * 10000 + 2 * 1000) * 10% + 300 = 1500PCX. All users have access to PCX mining revenue in each reward cycle according to the mining power they have. Users can use the PCX obtained through revenue to participate in elections or wait for the value of the inter-chain assets to go up to claim more mining power.

**Node rewards**

The system will calculate the rewards for each node according to the proportion of the total votes in each dividend cycle.



```
Node rewards = Votes of a node/ Total votes of all nodes * Total rewards
```


All nodes that are in the election and attract no less than 1 vote can access the rewards according to the number of votes they get over the total. A node that is forced off due to a previous offline penalty will not be able to receive rewards.

Note: Inter-chain assets participating in mining can be regarded as a virtual node which is yet part of the consensus, but could still receive rewards.

### Node prize pool

Rewards are reallocated after being issued to nodes:
1. 10% to the node account directly
2. 90% to the node prize pool

The node prize pool is a special account that no one knows the private key. The calculating logic of the prize pool account:


```
Pool account = Hash (node ​​account)
```


The interest receivable by users in the prize pool is not automatically allocated, but needs to be calculated on the proportion of the user's vote age to the total vote age of nodes when the user manually withdraws it.

### Vote age

Vote age equals coin age in deposits mining, which also holds true for voting mining. 

**Node vote age**

Each real node has the following attributes: total vote age, updated height of total vote age which is adjusted according to the total number of votes that nodes get when users cast or retract votes:


```
Total vote age = previous vote age +previous vote amount * (current height-previous updated height)

Updated height of total vote age = current height
```


**User vote age**

Users have the following attributes: user vote age, updated height of user vote age, which is adjusted according to the total number of votes that nodes get when users cast or retract votes:

```
User vote age = previous user vote age + previous user votes * (current height-previous updated height)

Updated height of user vote age = current height

```


**Voting interest**

If a user votes for a node, he or she can calculate the interest receivable after obtaining information about the node vote age and the current height:

```
Latest vote age = previous vote age + total voting amount * (current height-updated height)

Latest user vote age = user vote age + voting amount * (current height-updated height)

Interest receivable = latest user vote age / latest total vote age * amount in the prize pool
```


**Interest withdrawal**

A user withdraws his or her interest from the prize pool according to the proportion of the user's vote age to the total vote age of nodes. After receiving the interest, the following changes occur to the vote age:

```
Total node vote age = total node vote age-user vote age
User vote age = 0
```


### Node election

The validator nodes are elected every election cycle. Nodes that are qualified to stand the election must:
1. already be a candidate 
2. attract at least 1 vote 

Nodes that meet the above conditions can stand the validator node 
election and will be ranked according to the number of votes they get.

## Transaction interface

### Node registration 
* Parameters: node name
* Setting: anyone can submit a name which should be less than 16 bytes, and cannot be repeated.

Initialize the vote age of the node and set the initial status to “opt out of election”.

### Node update 
* Parameters: website, profile, block account ID, status (opt in election| opt out of election)
* Setting: only registered nodes can call an update; the website byte range is 4 to 24, allowing only numbers, uppercase and lowercase letters, and. , and the profile is less than 128 bytes.

If the number of opt-in nodes is less than the minimum number of validator nodes, then qualified nodes are not allowed to opt out. In other cases, the node information is updated directly.

### Trust node setting 
* Parameters: chain ID, profile, hot public key, cold public key
* Setting: Only the opt-in nodes can call the process; currently the chain is only for Bitcoin with correct public key format.

Update the node's trust settings because only after chains set the trust correctly can they be elected as trust nodes.

### Casting a vote
* Parameters: node account ID, increased votes, notes.
* Setting: anyone can call the process; the node account must exist and the increased amount of votes must be less than the user's PCX available balance; notes are less than 64 bytes.

If users cast their votes, accordingly the user and node's vote age should be updated.

### Retracting a vote
* Parameters: node account ID, retracted votes, notes.
* Setting: Only users who have voted for the node can call the 
process; the retracted amount must be less than or equal to the original voting; notes are less than 64 bytes.

After users retract their votes, a record is created and accordingly the user and node's vote age should be updated. The height of unfreezing the redeemed needs to be recalculated (redeeming the collaterals of a node or retracting votes). After the user retracts a node's votes, the redeemed assets yet still unfrozen cannot generate revenue.


### Unfreezing the redeemed
* Parameters: node account ID, index
* Setting: users who have the redeeming operation corresponding to the index can call the process, and the current height must be higher than the unlocked height.

The redeemed is to be converted into users’ balance with the record deleted.

### Withdrawing interest
* Parameter: asset ID
* Setting: Only users who hold the asset can call it.

Interest receivable which is calculated according to the above-mentioned method is to be issued to users’ balance from the prize pool.
