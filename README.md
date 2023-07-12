---
description: Change cosmos chain from permissionless to permissioned
---

# Make Permissioned Chain

\
1\. **Write a proposal**

Proposal example: This is the proposal to make the uploadAccess to limited addresses.

```json
// Proposal for Permission
{
   "title":"Change UploadAccess Permission",
   "description":"chnaging code/wasm upload access to only a few addresses",
   "changes":[
      {
         "subspace":"wasm",
         "key":"uploadAccess",
         "value":{
            "permission":"AnyOfAddresses",
            "addresses":[
               "wavehash1fttx53ed9jxhuerjv6xr2r6sfegjgchzygf74f",
               "wavehash1mg8wl2mjm4q4z3p3qrrvk9dfymc7qye5qf5x6r"
            ]
         }
      }
   ],
   "deposit":"2000000uwahax"
}
```

\


and save it as a json file.

\


**2**. **Submit a this proposal using command like this:**\
`wavehashd tx gov submit-proposal param-change proposalName.json --from keyName --chain-id chainIDName -b block`\
For example:&#x20;

{% code overflow="wrap" fullWidth="true" %}
```sh
wavehashd tx gov submit-proposal param-change proposalPermission.json --from validator --chain-id testnet-1 -b block
```
{% endcode %}

**3. Wait for the deposit time to over OR submit a deposit using this command like this:**\
wavehashd tx gov deposit proposalId AmountToBeDepositedwithDenom --from KeyName --chain-id chainIDName

For example:&#x20;

{% code overflow="wrap" %}
```bash
wavehashd tx gov deposit 1 8000000uwahax --from validator --chain-id testnet-1
```
{% endcode %}

\


**4. After deposit period is over, voting time is started. Now vote for the proposal using command like this:**\
wavehashd tx gov vote proposalId votingDecision(i.e. yes|no|abstain) --from keyName-- chain-id chainIDName

For example:&#x20;

```
wavehashd tx gov vote 1 yes --from validator --chain-id testnet-1
```

\


**5. Now, wait for the proposal voting time to pass**\


For More details Read Below OR visit this link: [https://github.com/CosmWasm/wasmd/blob/main/x/wasm/Governance.md](https://github.com/CosmWasm/wasmd/blob/main/x/wasm/Governance.md)



**# Governance**

This document gives an overview of how the various governance

proposals interact with the CosmWasm contract lifecycle. It is

a high-level, technical introduction meant to provide context before

looking into the code, or constructing proposals.



**## Proposal Types**

We have added 9 new wasm specific proposal types that cover the contract's live cycle and authorization:

```
*  StoreCodeProposal - upload a wasm binary
* `InstantiateContractProposal` - instantiate a wasm contract
* `MigrateContractProposal` - migrate a wasm contract to a new code version
* `SudoContractProposal` - call into the protected `sudo` entry point of a contract
* `ExecuteContractProposal` - execute a wasm contract as an arbitrary user
* `UpdateAdminProposal` - set a new admin for a contract
* `ClearAdminProposal` - clear admin for a contract to prevent further migrations
* `PinCodes` - pin the given code ids in cache. This trades memory for reduced startup time and lowers gas cost
* `UnpinCodes` - unpin the given code ids from the cache. This frees up memory and returns to standard speed and gas cost
* `UpdateInstantiateConfigProposal` - update instantiate permissions to a list of given code ids.
* `StoreAndInstantiateContractProposal` - upload and instantiate a wasm contract.
```

For details see the proposal type \[implementation]\(https://github.com/CosmWasm/wasmd/blob/master/x/wasm/types/proposal.go)



**## Wasmd Authorization Settings**

Settings via sdk \`params\` module:

\- \`**code\_upload\_access**\` - who can upload a wasm binary: \`**Nobody**\`, \`**Everybody**\`, \`**OnlyAddress**\`

\- \`**instantiate\_default\_permission**\` - platform default, who can instantiate a wasm binary when the code owner has not set it\
See \[params.go]\(https://github.com/CosmWasm/wasmd/blob/master/x/wasm/types/params.go)



**### Init Params Via Genesis**

{% code overflow="wrap" %}
```json
"wasm":{
   "params":{
      "code_upload_access":{
         "permission":"Everybody"
      },
      "instantiate_default_permission":"Everybody"
   }
},
```
{% endcode %}

The values can be updated via gov proposal implemented in the \`params\` module.

**Example to submit a parameter change gov proposal:**

{% code overflow="wrap" fullWidth="false" %}
```sh
wasmd tx gov submit-proposal param-change <proposal-json-file> --from validator 
--chain-id=testing -b block
```
{% endcode %}



**#### Content examples**

\* Disable wasm code uploads

```json
{
   "title":"Foo",
   "description":"Bar",
   "changes":[
      {
         "subspace":"wasm",
         "key":"uploadAccess",
         "value":{
            "permission":"Nobody"
         }
      }
   ],
   "deposit":""
}
```

\* Allow wasm code uploads for everybody

```json
{
   "title":"Foo",
   "description":"Bar",
   "changes":[
      {
         "subspace":"wasm",
         "key":"uploadAccess",
         "value":{
            "permission":"Everybody"
         }
      }
   ],
   "deposit":""
}
```

\* Restrict code uploads to a single address

```json
{
   "title":"Foo",
   "description":"Bar",
   "changes":[
      {
         "subspace":"wasm",
         "key":"uploadAccess",
         "value":{
            "permission":"OnlyAddress",
            "address":"wavehash1redfcv6yjt5re4tgfhvbuhjnmqqqq0fr2sh"
         }
      }
   ],
   "deposit":""
}
```

\* Set chain \*\*default\*\* instantiation settings to nobody

```json
{
   "title":"Foo",
   "description":"Bar",
   "changes":[
      {
         "subspace":"wasm",
         "key":"instantiateAccess",
         "value":"Nobody"
      }
   ],
   "deposit":""
}
```

\* Set chain \*\*default\*\* instantiation settings to everybody

```json
{
   "title":"Foo",
   "description":"Bar",
   "changes":[
      {
         "subspace":"wasm",
         "key":"instantiateAccess",
         "value":"Everybody"
      }
   ],
   "deposit":""
}
```

**### Enable gov proposals at \*\*compile time\*\*.**

As gov proposals bypass the existing authorization policy they are disabled and require to be enabled at compile time.

```
-X github.com/CosmWasm/wasmd/app.ProposalsEnabled=true - enable all x/wasm governance proposals (default false)
-X github.com/CosmWasm/wasmd/app.EnableSpecificProposals=MigrateContract,UpdateAdmin,ClearAdmin - enable a subset of the x/wasm governance proposal types (overrides ProposalsEnabled)
```

The \`ParamChangeProposal\` is always enabled.

\## CLI

```
wasmd tx gov submit-proposal [command]
Available Commands:
wasm-store               Submit a wasm binary proposal
instantiate-contract     Submit an instantiate wasm contract proposal
migrate-contract         Submit a migrate wasm contract to a new code version proposal
set-contract-admin       Submit a new admin for a contract proposal
clear-contract-admin     Submit a clear admin for a contract to prevent further migrations proposal
```

