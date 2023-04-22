# Tutorial for Carbon retirement on Celo

In this Tutorial we will learn how retire carbon credits on Celo, using Celo Composer and the Toucan SDK.
What we are going to do more in detail is the following: we will first get NCTs. These are carbon pool tokens, which means, we can get them at a DEX like [Ubeswap](https://ubeswap.org). Then we will redeem them for TCO2s which are tokenized Carbon Certificate. They hold all attributes about the project that created them. These tokens can then be retired and you can receive a certificate for that. In the end we will also learn how to query the [subgraph](https://thegraph.com/hosted-service/subgraph/toucanprotocol/alfajores), when building a page to show the users retirements.

For everybody who is new to the Voluntary Carbon Markets, let me quickly explain, why we need the pool tokens. Carbon Credits vary in their attributes depending on the project that issued them like country, year off issuance etc... Here `TCO2-` a general term for fungible tokenized carbon credits serves as a prefix, followed by an information-rich name that includes the registry of origin, the project, the vintage, and so on. Actually the symbol of TCO2s already gives you some information about the project. E.g. `TCO2-VCS-<projectId>-YYYY`, shows you that the credit comes from a project from the Verified Carbon Standard crediting program, the projectId and the vintage starts YYYY/01/01 and ends YYYY/12/31. If the vintage's start and end are not both exactly aligned to year boundaries one year apart then the format is instead `TCO2-VCS-<projectId>-YYYYMMDD` where YYYYMMDD is the vintage start date.

Coming back to why we have two kind of tokens: As a solution to the resulting liquidity problem, Toucan has created a pool infrastructure where tokens that fulfil certain criteria can be deposited in exchange for pool tokens.

1. Install Celo-Composer
2. Retire Carbon Credits: Either use the Toucan SDK or interact directly with the Toucan Contracts

   2. Install the SDK  
      2.1. Install the SDK  
      2.2. Get Toucan Client  
      2.3. Redeem Tokens form a PoolContract (e.g. NCT)  
      2.4. Retire TCO2s  
      2.5. Get a certificate  
      2.6. Interact with Toucan's Contracts

3. Creating a list of our retirements

## 1. Install [Celo-Composer](https://docs.celo.org/blog/tutorials/building-our-first-smart-contract-web-dapp-with-celo-composer)

We will use [Celo-Composer](https://docs.celo.org/blog/tutorials/building-our-first-smart-contract-web-dapp-with-celo-composer) to quick-start our web3 application. It already comes with several wallet integrations using [rainbow-kit](https://www.rainbowkit.com), [wagmi](https://wagmi.sh) for easy interactions with the blockchain and [tailwind](https://tailwindcss.com).

```
npx @celo/celo-composer create
```

To create a simple example project we just chose the default, except for `Choose smart-contract framework:`, there we choose `none` as we won't need to develop smart contracts if we only want to use the Toucan SDK.

![image of the options selected](./assets/composer-selection.jpg)

Great. Now let's open the project in our favorite IDE (e.g. VS Code).

First navigate into the react-app.

```
cd packages/react-app/
```

Here install all dependencies with

```
npm i
```

or

```
yarn
```

---

And finally, let's start the App and see if everything is running.

```
npm run dev
```

or

```
yarn run dev
```

---

## 2. Retire Carbon Credits

Next we are going to retire carbon credits on Celo using the [Toucan SDK](https://github.com/ToucanProtocol/toucan-sdk). The Toucan SDK provides you with tools to simply implement carbon retirements into your app with just a few lines of code. If also provides some pre-defined subgraph queries but offers you the freedom to create any query your heart desires to get all the info about all retirements and tokens.

## 2.1. Install the SDK

Add the Toucan SDK.

```
npm i toucan-sdk
```

or

```
yarn add toucan-sdk
```

## 2.2. Get Toucan Client

When using the Toucan SDK we want to first instantiate the ToucanClient and set a signer & provider to interact with our infrastructure. We can use the signer & provider from the wagmi library. For interacting with The Graph, no provider or signer is needed though. But we will talk about that [later](#3-creating-a-list-of-our-retirements).

So in the `index.tsx` file we will add the imports to the top:

```typescript
import ToucanClient from "toucan-sdk";
import { useProvider, useSigner } from "wagmi";
```

And this part into our function body

```typescript
const provider = useProvider();
const { data: signer, isError, isLoading } = useSigner();

const toucan = new ToucanClient("alfajores", provider);
signer && toucan.setSigner(signer);
```

<details>
<summary>Our code will look like this now:</summary>

```javascript
import { ToucanClient } from "toucan-sdk";
import { useProvider, useSigner } from "wagmi";

export default function Home() {
  const provider = useProvider();
  const { data: signer, isError, isLoading } = useSigner();

  const toucan = new ToucanClient("alfajores", provider);
  signer && toucan.setSigner(signer);

  return (
    <div>
      <div className="h1">
        There we go... a canvas for your next Celo project!
      </div>
    </div>
  );
}
```

</details>

## 2.3. Redeem Tokens form a PoolContract (e.g. NCT)

To retire Carbon Credits need pool tokens (e.g.NCTs) or TCO2. We can get them from the [Toucan Faucet](https://faucet.toucan.earth/). We should get NCT, as theses are the tokens we can buy in an exchange like Ubeswap. We will only have TCO2 tokens, if we tokenzied carbon credits yourself or if we have already redeemed the NCTs for TCO2s. So this example will start with NCTs.

| :herb: Get some Nature Carbon Tonnes (NCT) before we continue form the [Toucan Faucet](https://faucet.toucan.earth/) . Make sure we have CELO to pay the gas fee for the withdrawel, we can get some from the [Celo Faucet](https://faucet.celo.org/alfajores). :herb: |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

We can auto-redeem the Pool tokens with [`redeemAuto2`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts#redeemauto2), where they are exchanged for the lowest ranking TCO2s. Auto-redeem also returns the addresses of the redeemed TCO2s, which we need for the next step. As arguments for the function, we will need the current address of the pool symbol, that we want to retire, like "NCT". We will also need to input the amount of tokens we wish to retire. We can read more upon the functions in our [documentation](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/pool-contracts).

If we want to choose the TCO2s that we want to retire, we can get a list of all TCO2 with `getScoredTCO2s` and then select the ones we prefer. Currently scored TCO2 means, that the tokens are sorted by year with scoredTokens[0] being the lowest. When getting the highest TCO2, make sure that the balance of the token is not 0. When we've chosen then ones we want to redeem our pool tokens for (we can choose several) we can redeem them with `redemMany`. For this Toucan Protocol takes fees. We can calculate the fee beforehand with `calculateRedeemFees`.

But today we stay simple with `redeemauto2`:

```typescript
toucan.redeemAuto2("NCT", parseEther("1"));
```

Now let's put that code in a function and add a button to trigger it, so we can see it in action!! We also want to store the return value, the TCO2 address in a variable, as we will want to use it in the next step.

<details>
<summary>Our code should look like this now: </summary>

```typescript
import { parseEther } from "ethers/lib/utils.js";
import { ToucanClient } from "toucan-sdk";
import { useProvider, useSigner } from "wagmi";

export default function Home() {
  const provider = useProvider();
  const { data: signer, isError, isLoading } = useSigner();
  const toucan = new ToucanClient("alfajores", provider);
  signer && toucan.setSigner(signer);
  // we will store our return value here
  const [tco2address, setTco2address] = useState("");

  const redeemPoolToken = async (): Promise<void> => {
    const redeemedTokenAddress = await toucan.redeemAuto2(
      "NCT",
      parseEther("1")
    );
    redeemedTokenAddress && setTco2address(redeemedTokenAddress[0].address);
  };

  return (
    <div>
      <div className="h1">
        <div>
          <button
            className="group relative flex w-full justify-center rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
            onClick={() => redeemPoolToken()}
          >
            <span className="absolute inset-y-0 left-0 flex items-center pl-3"></span>
            {"Redeem Tokens"}
          </button>
        </div>{" "}
      </div>
    </div>
  );
}
```

</details>

But we will probably get an error like this:
![Error: No signer yet](./assets/error-no-signer.jpeg)

And that makes sense, because we are not yet connected with our wallet. So let's do that.

| :eyeglasses: Try it out and check the transaction on [Celoscan](https://alfajores.celoscan.io) :eyeglasses: |
| ----------------------------------------------------------------------------------------------------------- |

## 2.4. Retire TCO2s

After **_Redeeming_** our pool tokens for TCO2s we will be able to retire them. We can only retire **TCO2s** tokens. We can either choose to simply [`retire`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retire) or if we would like to retire for a third party use the [`retireFrom`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retirefrom) function. Lastly we can also already get a certificate created with [` retireAndMintCertificate`](https://docs.toucan.earth/toucan/dev-resources/smart-contracts/tco2#retireandmintcertificate). - [Example ABI](https://github.com/ToucanProtocol/contracts/blob/main/artifacts/staging/celo-alfajores/ToucanCarbonOffsets.json)

The first thing we will have to do, will be to get the address of our TCO2 token. We will have saved that as return value form `autoRedeem2` or from when we selected the tokens. And now we can retire our token, with adding this line of code to our redeem function.

```typescript
await toucan.retire(parseEther("1.0"), tco2Address);
```

Let's create a second function called `retirePoolToken` as well as a button for the retirement process.

<details>
<summary>Our code should look like this now: </summary>

```tsx
import { parseEther } from "ethers/lib/utils.js";
import { useState } from "react";
import { ToucanClient } from "toucan-sdk";
import { useProvider, useSigner } from "wagmi";

export default function Home() {
  const provider = useProvider();
  const { data: signer, isError, isLoading } = useSigner();
  const toucan = new ToucanClient("alfajores", provider);
  signer && toucan.setSigner(signer);
  const [tco2address, setTco2address] = useState("");

  const redeemPoolToken = async (): Promise<void> => {
    const redeemedTokenAddress = await toucan.redeemAuto2(
      "NCT",
      parseEther("1")
    );
    redeemedTokenAddress && setTco2address(redeemedTokenAddress[0].address);
  };

  const retireTco2Token = async (): Promise<void> => {
    tco2address.length && (await toucan.retire(parseEther("1.0"), tco2address));
  };

  return (
    <div>
      <div className="h1">
        <div>
          <button
            className="group relative flex w-full justify-center rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
            onClick={() => redeemPoolToken()}
          >
            <span className="absolute inset-y-0 left-0 flex items-center pl-3"></span>
            {"Redeem Tokens"}
          </button>
          <button
            className="group relative flex w-full justify-center rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
            onClick={() => retireTco2Token()}
          >
            <span className="absolute inset-y-0 left-0 flex items-center pl-3"></span>
            {"Retire Tokens"}
          </button>
        </div>{" "}
      </div>
    </div>
  );
}
```

</details>

# 3. Creating a list of our retirements

In the last step, let's create a list showing our retirements. First we will create a new `list.tsx` page.

There we need the ToucanClient. But, we won't need a provider or signer for querying the subgraph.

```typescript
const toucan = new ToucanClient("alfajores");
```

The Toucan SDK has several pre-defined queries to get data from the subgraph, but we can also create our customized query with `toucan.fetchCustomQuery()`. We can check all schemes, create and test our query in the [playground](https://thegraph.com/hosted-service/subgraph/toucanprotocol/alfajores) of the Toucan Subgraph.

But now we will use one of the predefined queries to get a list of our retirements. Remember that an address always needs to be lower case for querying, otherwise we won't get any results.

```typescript
await toucan.fetchUserRetirements(address?.toLowerCase());
```

Now let's add some code to display out retirements in a table.

<details>
<summary>And now our code should look like this now: </summary>

```tsx
import { useEffect, useState } from "react";
import ToucanClient, { fetchUserRetirementsResult } from "toucan-sdk";
import { useAccount } from "wagmi";

export default function Sdk() {
  const toucan = new ToucanClient("alfajores");
  const { address } = useAccount();

  const [retirements, setRetirements] = useState<fetchUserRetirementsResult[]>(
    []
  );

  const getUserRetirements = async () => {
    const result =
      address && (await toucan.fetchUserRetirements(address?.toLowerCase()));
    result && setRetirements(result);
  };

  useEffect(() => {
    !retirements.length && getUserRetirements();
  });

  return (
    <div>
      <div className="flex min-h-full items-center justify-center px-4 py-12 sm:px-6 lg:px-8">
        <div className="w-full space-y-8">
          <div className="">
            <h2 className="mt-6 text-center text-3xl font-bold tracking-tight text-blue-900">
              My Retirements{" "}
            </h2>
          </div>
          <div className="flex justify-center"></div>

          {retirements.length ? (
            <div className="relative overflow-x-auto">
              <table className="w-full text-sm text-left text-blue-500 dark:text-blue-400">
                <thead className="text-xs text-blue-700 uppercase bg-blue-50 dark:bg-blue-700 dark:text-blue-400">
                  <tr>
                    <th className="px-6 py-3">Token name</th>
                    <th className="px-6 py-3">Token symbol</th>
                    <th className="px-6 py-3">Certificate ID</th>
                    <th className="px-6 py-3">Creation Transaction</th>
                  </tr>
                </thead>
                <tbody>
                  {retirements.map((item) => {
                    return (
                      <tr
                        className="bg-white border-b dark:bg-blue-800 dark:border-blue-700"
                        key={item.id}
                      >
                        <td className="px-6 py-4 font-medium text-blue-900 whitespace-nowrap dark:text-white">
                          {item.token.name}
                        </td>
                        <td className="px-6 py-4">{item.token.symbol}</td>
                        <td className="px-6 py-4">{item.certificate?.id}</td>
                        <td className="px-6 py-4">{item.creationTx}</td>
                      </tr>
                    );
                  })}
                </tbody>
              </table>
            </div>
          ) : (
            <div>You don't have retired any carbon credits yet</div>
          )}
        </div>
      </div>
    </div>
  );
}
```

</details>
