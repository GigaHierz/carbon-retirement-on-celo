# Tutorial for Carbon retirement on Celo

In this Tutorial we will learn how to retire carbon credits on Celo, using Celo Composer and the Toucan SDK.

1. Install [Celo-Composer](https://docs.celo.org/blog/2022/02/21/introduction-to-celo-progressive-dappstarter)

2. Retire carbon credits with Toucans SDK  
   2.1. Install the SDK  
   2.2. Get Toucan Client  
   2.3. Redeem Tokens form a PoolContract (NCT/BCT)  
   2.4. Retire your TCO2s  
   2.5.
   2.4. Find more functionalities

## 1. Install [Celo-Composer](https://docs.celo.org/blog/2022/02/21/introduction-to-celo-progressive-dappstarter)

To get started building out app, we will use the [Celo-Composer](https://docs.celo.org/blog/2022/02/21/introduction-to-celo-progressive-dappstarter) that already comes in with the wallet integration, wagmi hooks and tailwind for styling.

## 2. Retire carbon credits with Toucans SDK

Using the Toucan SDK to retire Carbon Credits

### 2.1. Install the SDK

```
npm i toucan-sdk
```

or

```
yarn add toucan-sdk
```

#### 2.2. Get Toucan Client

Instantiate the ToucanClient and set a signer & provider to interact with our infrastructure. We can use the signer & provider from the wagmi library

```typescript
import { ToucanClient } from "toucan-sdk";
import { useProvider, useSigner } from "wagmi";
const provider = useProvider();
const { data: signer, isError, isLoading } = useSigner();


const toucan = new ToucanClient("alfajores", provider, signer);
```

### 2.3. Redeem Tokens form a PoolContract (NCT/BCT)

First you should get some pool tokens (e.g.NCTs) or TCO2 from the [Toucan Faucet](https://faucet.toucan.earth/). If you have NCTs, you will first have to redeem them for TCO2s.

You will have to decide if you want

- to auto-redeem them with [`redeemAuto2`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts#redeemauto2), where they are exchanged for the lowest ranking TCO2s. Auto-redeem returns the addresses of the redeemed TCO2s, which you need for the next step. As Arguments for the function, you will need the current address of the pool token, that you want to retire, like NCT, which you can find on [this page](https://app.toucan.earth/contracts) with all deployed Toucan contracts. You will also need to input the amount of tokens you wish to retire. You can read more upon them in our [documentation](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts).

```typescript
const nct = toucan.getPoolContract("NCT");
nct.redeemAuto2(parseEther("1.0"));
```

- or you can get a list of all TCO2 with [`getScoredTCO2s`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts#getscoredtco2s) and then select the ones you prefer. When getting the highest TCO2, make sure that the balance of the token is not 0. When choosing and redeem them with [`redemMany`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts#redeemmany). For this Toucan Protocol takes [fees](https://docs.toucan.earth/toucan/pool/protocol-fees). You can calculate the fee beforehand with [`calculateRedeemFees`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts#calculateredeemfees).

```typescript
const nct = toucan.getPoolContract("NCT");
nct.tokenBalances(tco2Address);
```

### 2.4. Retire your TCO2s

After **_Redeeming_** your pool tokens for TCO2s you will be able to retire them. You can only retire **TCO2s** tokens. You can either choose to simply [`retire`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retire) or if you would like to retire for a third party use the [`retireFrom`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retirefrom) function. Lastly you can also already get a certificate created with [` retireAndMintCertificate`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retireandmintcertificate). - [Example ABI](https://github.com/ToucanProtocol/contracts/blob/main/artifacts/staging/celo-alfajores/ToucanCarbonOffsets.json)

The first thing you will have to do, will be to get the address of your TCO2 token. You will have saved that as return value form `autoRedeem2` or from when you selected the tokens.

```typescript
const tco2 = toucan.getTCO2Contract(tco2Address);
tc02.retire(parseEther("1.0"));
```

### 2.3. Get a certificate

If you want to create certificates afterwards, you can als just do that with [`mintCertificate`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/retirement-certificates#mintcertificate) from the RetirementCertificates Contract.

### 2.4. Interact with Toucan's Contracts

You can find all deployed contracts on the [Toucan App](https://app.toucan.earth/contracts).

Now you can interact with them.

```typescript
const nct = toucan.getPoolContract("NCT");
const tco2 = toucan.getTCO2Contract(tco2Address);
const registry = toucan.getRegistryContract();
const remainingTCO2 = await nct.tokenBalances(tco2Address);
```

### 2.5. Get a certificate

If you want to create certificates afterwards, you can als just do that with [`mintCertificate`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/retirement-certificates#mintcertificate) from the RetirementCertificates Contract.

```typescript
const nct = toucan.getPoolContract("NCT");
const tco2 = toucan.getTCO2Contract(tco2Address);
const registry = toucan.getRegistryContract();
const remainingTCO2 = await bct.tokenBalances(tco2Address);
```

### 2.6. Find more functionalities

Please check the table for link to the docs for each specific contract. You can also find all current deployed contacts on the [Toucan App](https://app.toucan.earth/contracts) and in our [GitHub repo](https://github.com/ToucanProtocol/contracts/tree/main/artifacts).

| Contract                                                                                                          | Function                   | Description                                                                                                                                                                                                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [PoolContracts (NCT/BCT)](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts)          | `redeemAuto`               | Automatically redeems pool tokens for underlying TCO2 tokens. This is 1:1 and doesn't incur fees. But you will receive what are considered lower quality TCO2s based on an arbitrary index called scoredTCO2s (to be explained further down below). The user's pool tokens get burnt within the process. |
| [PoolContracts (NCT/BCT)](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts)          | `redeemAuto2`              | `redeemAuto2` acts very similar to `redeemAuto` but it also returns arrays of the redeemed TCO2s. This uses more gas but it is going to be more optimal to use by other on-chain contracts.                                                                                                              |
| [PoolContracts (NCT/BCT)](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts)          | `getScoredTCO2s`           | Returns an array of ranked TCO2 addresses. scoredTCO2s[0] being the lowest rank, the rank is then ascending. You want to call this function if you are planning to retire specific TCO2s. Or specifically TCO2s with a higher ranking.                                                                   |
| [PoolContracts (NCT/BCT)](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts)          | `redeemMany`               | Selectively redeems pool tokens for underlying TCO2 tokens. This is 1:1 minus fees. The user's pool tokens get burnt within the process.                                                                                                                                                                 |
| [Toucan Carbon Offset](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2)                       | `retire`                   | You retire a given amount of CO2 tons. The tokens get burnt and this achieves the offset. This also emits a Retired event.                                                                                                                                                                               |
| [Toucan Carbon Offset](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2)                       | `retireFrom`               | Achieves similar functionality as retire(), but instead of retiring from the callers address, it does so from the given address. This allow for pools or third party contracts to retire for the user.                                                                                                   |
| [Toucan Carbon Offset](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2)                       | `retireAndMintCertificate` | Just as retire() this retires an amount of TCO2, but after the Retired event is emited this function mints a certificate passing the given retirementEventId.                                                                                                                                            |
| [Retirement Certificates](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/retirement-certificates) | `mintCertificate`          | Mints a new RetirementCertificates NFT based on existent Retired events. The function can either be called by a valid TCO2 contract (in its retireAndMintCertificate() function) or by a user who owns Retired events.                                                                                   |
