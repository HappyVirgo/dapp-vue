<template>
  <div id="app">
    <span class="button" v-if="!web3" v-on:click="connectWeb3">Connect</span>
    <span v-else>Connected Address: {{address}}</span>
    <div>
      <span class="button" v-on:click="onBuy">Buy</span>
      <input v-model="amount" placeholder="Please input amount" :class="(validateValue && 'error') + ' input'"  />
      <p v-if="pending">Pending now...</p>
      <div v-else>
        <div style="padding: 20px" v-if="Object.keys(eventValue).length">
          <p>Buyer: {{buyer}}</p>
          <p>Amount: {{buyAmount}} ether</p>
          <p>Date: {{date}}</p>
        </div>
      </div>
      <p v-if="error" class="errorMessage">{{error}}</p>
    </div>
  </div>
</template>

<script>
import Web3 from 'web3'
import Constants from './constants'
// import Contract from '../artifacts/contracts/MyContract.sol/MyContract.json';

export default {
  name: 'App',
  data(){
      return {
        open: false,
        web3: null,
        deployedAddress: '',
        eventValue: {},
        pending: false,
        address: '',
        validateValue: '',
        amount: '',
        buyAmount: '',
        error: null,
        date: ''
      }
  },
  methods:{
    async connectWeb3() {
      if(typeof window.ethereum !== undefined) {
        this.web3 = new Web3(window.ethereum);
        try{
          await window.ethereum.enable();
          this.address = this.web3.currentProvider.selectedAddress;
        } catch(e) {
          console.log("error occured", e);
        }
        console.log(this.web3.eth.accounts);
      }
    },
    async onBuy() {
      if(!this.web3) {
        alert('Please connect your wallet')
        this.connectWeb3()
      }
      if(this.amount === '' || !parseFloat(this.amount)) {
        console.log('here')
        this.validateValue = 'Please input value'
        return;
      }
      let myContract = new this.web3.eth.Contract(Constants.ABI, Constants.address)
      this.pending = true
      console.log('amount', this.amount.toString(), Constants.address);
      try{
        await myContract.methods.buy()
        .send({
            from: this.address, 
            value: Web3.utils.toWei(this.amount.toString(), 'ether')
          }, 
          (err, result) => {
            if (err) {
              this.error = "Something went wrong"
              console.log('err', err)
              this.pending = false
            } else {
              console.log(result)
            }
          }
        );
        await myContract.events.Buy({
          fromBlock: 0
        }, async (error, event) => { 
          console.log('result', event)
         })
        .on('data', async (event) => {
          console.log('result', event)
          this.buyer = event.returnValues.buyer
          this.buyAmount = Web3.utils.fromWei(event.returnValues.amount.toString(), 'ether')
          let block = await this.web3.eth.getBlock(event.blockNumber)
          let date = new Date(block.timestamp * 1000);
          this.date = date.toUTCString()
          this.eventValue = event
          this.pending = false
        })
        .on('changed', function(event){
          console.log('event', event)
            // remove event from local database
        })
        .on('error', console.error);
      } catch(e) {
        console.log("error", e.message)
        this.error = "Something went wrong"
        this.pending = false;
      }
    }
  }
}
</script>

<style>
body {
  margin: 0;
}
table {
  border-collapse: collapse;
  width: 80%;
  margin: auto;
}

th, td {
  text-align: left;
  padding: 8px;
}

tr:nth-child(even){background-color: #f2f2f2}

th {
  background-color: #7fcdc1;
  color: white;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 20px;
  height: 100vh;
}
.button {
  padding: 10px 16px;
  border-radius: 10px;
  background: #7fcdc1;
  color: white;
  cursor: pointer;
}
.input {
  padding: 10px 6px;
  background: #ededed;
  outline: none;
  border: none;
  margin-left: 10px;
}
.error.input {
  border: 1px solid red;
}
.errorMessage {
  color: red;
}
</style>
