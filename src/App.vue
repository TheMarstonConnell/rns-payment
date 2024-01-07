<script setup lang="ts">
import { ref, type Ref } from 'vue'
import gfm from '@bytemd/plugin-gfm'
import 'bytemd/dist/index.css'
import {type IWalletHandler, WalletHandler, RnsHandler} from '@jackallabs/jackal.js'
import {SigningStargateClient} from '@cosmjs/stargate'

const wallet: Ref<any> = ref({})
const stargateClient: Ref<any> = ref({})
const rns: Ref<any> = ref({})
const walletActive : Ref<boolean> = ref(false)


const mSignerChain = 'jackal-1'
const mainnet = {
  signerChain: mSignerChain,
  enabledChains: [mSignerChain],
  queryAddr: 'https://grpc.jackalprotocol.com',
  txAddr: 'https://rpc.jackalprotocol.com',
  chainConfig: { // mainnet chain config
      chainId: mSignerChain,
      chainName: 'Jackal',
      rpc: 'https://rpc.jackalprotocol.com',
      rest: 'https://api.jackalprotocol.com',
      bip44: {
        coinType: 118
      },
      coinType: 118,
      stakeCurrency: {
        coinDenom: 'JKL',
        coinMinimalDenom: 'ujkl',
        coinDecimals: 6
      },
      bech32Config: {
        bech32PrefixAccAddr: 'jkl',
        bech32PrefixAccPub: 'jklpub',
        bech32PrefixValAddr: 'jklvaloper',
        bech32PrefixValPub: 'jklvaloperpub',
        bech32PrefixConsAddr: 'jklvalcons',
        bech32PrefixConsPub: 'jklvalconspub'
      },
      currencies: [
        {
          coinDenom: 'JKL',
          coinMinimalDenom: 'ujkl',
          coinDecimals: 6
        }
      ],
      feeCurrencies: [
        {
          coinDenom: 'JKL',
          coinMinimalDenom: 'ujkl',
          coinDecimals: 6,
          gasPriceStep: {
            low: 0.002,
            average: 0.002,
            high: 0.02
          }
        }
      ],
      features: []
  }
}

async function connectWallet() {

  const walletConfig = {
    selectedWallet: 'keplr',
    signerChain: mainnet.signerChain,
    enabledChains: mainnet.enabledChains,
    queryAddr: mainnet.queryAddr,
    txAddr: mainnet.txAddr,
    chainConfig: mainnet.chainConfig
  }

  // Hooking up the wallet to your app
  wallet.value = await WalletHandler.trackWallet(walletConfig)
  rns.value = await RnsHandler.trackRns(wallet.value)
  stargateClient.value = await SigningStargateClient.connectWithSigner(mainnet.txAddr, wallet.value.getSigner());
  walletActive.value = true
}


const name = ref('')
const amount = ref(10)
const selectedToken = ref("JKL")


const loading = ref(false)

async function pay() {
  const rnsName = await rns.value.findSingleRns(name.value)

  const recipient = rnsName.value
  const amt = {
    denom: "ujkl",
    amount: (Math.trunc(amount.value * 1_000_000)).toString(),
  };
  const fee = {
    amount: [
        {
        denom: "ujkl", 
        amount: "0.02",
        },
    ],
    gas: "82000",
  };

  stargateClient.value.sendTokens(wallet.value.getJackalAddress(), recipient, [amt], fee, "Sent from RNS payment gateway!").then((res: any) => {
    console.log(res)
    alert("Sent!")
  }).catch(() => {
    alert("Failed to send!")
  })
  
}


const isDisabled = function() : boolean {
  return !walletActive.value || loading.value
}

const nameChosen = function() : boolean {
  return name.value.length == 0
}

</script>

<template >
  <main>
    <h1>RNS Payements</h1>
    <button type="button" @click="connectWallet">{{ (walletActive == true) ? "Connected" : "Connect Wallet"}}</button>
    <div v-if="walletActive">
      <input placeholder="exampleName" class="rns" v-model="name"  >
      <span class="rm">.rns</span>
    <input class="tokens" v-model="amount"  >   
    <select v-model="selectedToken">
      <option value="JKL">JKL</option>
    </select>
    <button @click="pay" :disabled="nameChosen()">Pay</button>
    </div>
    
  </main>
</template>

<style>
#app {
  max-width: 1280px;
  margin: 0 auto;
  font-weight: normal;
  /* padding-top: 10px; */
}

.rm {
  margin-right: 10px;
}

h1 {
  margin-top: 0px;
}

.bytemd {
  width: 60vw;
  height: calc(100vh - 200px);
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 30px;
  margin-top: 30px;
}

.gateway-link,.file-title {
  cursor: pointer;
}

.gateway-link:hover {
  text-decoration: underline;
}

.file-title:hover {
  text-decoration: underline;
}

button {
  margin-left: 20px;
  margin-right: 20px;
  border-width: 1px;
  border-style: solid;
  border-color: black;
}

button:disabled {
  cursor: auto;
  border-color: black;
}

.filename {
  box-sizing: border-box;
  padding: 5px;
  font-size: 16px;
  border-width: 1px;
  border-radius: 8px;
  height: 40px;
  padding-left: 10px;
}

.bytemd-body {
  text-align: left;
}

.files {
    display: grid;
    grid-auto-flow: row; 
    grid-template-columns: 1fr 1fr 1fr; 
    gap: 10px 10px; 
}

.card > h1 {
  font-size: 1.6em;
}

.card {
  border-radius: 5px;
  border-style: solid;
  padding: 10px;
  overflow: hidden;
}

</style>
