<script setup>
import { ref, onMounted } from 'vue'
import { OKXUniversalProvider } from "@okxconnect/universal-provider";
import { OKXUniversalConnectUI, THEME } from "@okxconnect/ui";

let account;
const okxUniversalProvider = ref(null);
const universalUi = ref(null);

let okxUniversalConnectUI;

onMounted(async () => {
  universalUi.value = await OKXUniversalConnectUI.init({
    dappMetaData: {
      icon: "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png",
      name: "OKX Connect Demo"
    },
    actionsConfiguration: {
      modals: 'error',
    },
    language: "en_US",
    uiPreferences: {
      theme: THEME.LIGHT
    },
  });
  console.debug("okxUniversalProvider init success");
});

const handleClick = async () => {
  console.debug("handleClick");
  // if (!okxUniversalProvider.value.connected()) {
  if (!universalUi.value.connected()) {
    var session = await universalUi.value.openModal({
      namespaces: {
        eip155: {
          chains: ["eip155:1"],
          defaultChain: "1"
        }
      }
    })

    console.debug("connected session: ", session);
  }

  account = await universalUi.value.request({ "method": "eth_requestAccounts" })
  console.debug("connected account: ", account);
  // sign
  const timestamp = Math.floor(Date.now() / 1000).toString();
  const sign_message = `Welcome to AstroTrove___${timestamp}`;

  let data = {
    "method": "personal_sign",
    "params": [
      sign_message,
      account
    ]
  }

  let personalSignResult = await universalUi.value.request(data)
  alert(JSON.stringify(personalSignResult));
}


const connectAptos = async () => {
  okxUniversalConnectUI = await OKXUniversalConnectUI.init({
    dappMetaData: {
      icon: "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png",
      name: "OKX Connect Demo"
    },
    actionsConfiguration: {
      modals: "all"
    },
    language: "en_US",
    uiPreferences: {
      theme: THEME.LIGHT
    },
  });

  var session = await okxUniversalConnectUI.openModal({
    namespaces: {
      aptos: {
        chains: ["aptos:mainnet", // aptos mainnet
          // "movement:testnet",// movement testnet
        ],
      }
    }
  })

  console.log("session: ", session);
}

const sendAptos = async () => {
  await connectAptos();

  const transactionData = {
    arguments: [toAddress, `1000}`], // aptos deciaml 10^8
    function: '0x1::coin::transfer',
    type: 'entry_function_payload',
    type_arguments: ['0x1::aptos_coin::AptosCoin'],
  };


  let provider = new OKXAptosProvider(okxUniversalConnectUI)

  console.log("sending transaction...");

  await provider.signAndSubmitTransaction(transactionData, "aptos:mainnet")

}

</script>

<template>
  <div class="h-[100px] flex items-center justify-center">
    <!-- <button @click="handleClick"
      class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg transform transition-all duration-300 hover:scale-105 active:scale-95">
      Connect and Sign in
    </button> -->
    <button @click="sendAptos"
      class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg transform transition-all duration-300 hover:scale-105 active:scale-95">
      Connect Aptos and Transfer to self
    </button>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
