<script>

import metahubjs from 'metahubjs';
import { Api, JsonRpc } from 'eosjs';

const appName = 'Your App Name';
const chainId = 'aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906';
const rpcUrl = 'https://eos.greymass.com';

// connect
const connected = await metahubjs.connect();
if (!connected) {
  throw new Error('no metahub');
}

const network = { chainId };
const rpc = new JsonRpc(rpcUrl, { fetch });
const api = metahubjs.eos(network, Api, { rpc });


export default {
  data() {
    return {
      islogin: false,
      account: null,
      trxid: '',
      error: '',
    }
  },
  mounted() {
    if (metahubjs.identity) {
      this.account = metahubjs.identity.accounts[0];
      this.islogin = true;
    }
  },
  methods: {
    async clickLogin() {
      try {
        let id = await metahubjs.login({ appName, chainId });
        if (id) {
          this.account = id.accounts[0];
          this.islogin = true;
        }
        this.error = '';
      } catch (e) {
        this.error = e.message;
      }
    },
    async clickLogout() {
      await metahubjs.logout();
      this.account = null;
      this.islogin = false;
    },
    async clickTransfer() {
      try {
        const result = await api.transact({
              actions: [{
                account: 'eosio.token',
                name: 'transfer',
                authorization: [{
                  actor: this.account.name,
                  permission: this.account.authority,
                }],
                data: {
                  from: this.account.name,
                  to: 'metahubpower',
                  quantity: '0.1000 EOS',
                  memo: '',
                },
            }] 
          }, {
            blocksBehind: 3,
            expireSeconds: 30,
          }
        );
        this.trxid = result.transaction_id;
        this.error = '';
      } catch (e) {
        this.error = e.message;
      }

    },
  }
}

</script>

<template>
    <div class="nav">
        <div v-if="islogin">
          {{ account.name }}  <button type="button" @click="clickLogout">logout</button>
        </div>
        <button type="button" v-else @click="clickLogin">login</button>
    </div>
    <div class="apps" v-if="islogin">
      <button type="button" @click="clickTransfer">test transfer 0.1 EOS</button>
      <div class="trxid" v-if="trxid">transfer success, trxid: {{ trxid }}</div>
    </div>
    <div class="error" v-if="error">
      error: {{ error }}
    </div>
</template>

<style>
button {
  padding: 5px 10px;
  margin-right: 15px;
}
.apps {
  margin-top: 20px;
}
.apps .trxid {
  margin-top: 15px;
}
.error {
    margin-top: 20px;
    color: red;
}

</style>
