<template>
  <div>
    <h2>{{this.$route.params.id}}</h2>
    <VueQrcode class="qrcode" id="tdsfgdsfgs" :text="this.$route.params.id" :size="128"></VueQrcode>
    <p>I'm still building up an address-transaction database inlcluding the pending transactions. Until then, old transactions are coming from <a href="https://etherscan.io/">Etherscan</a> (They are awesome), but new transactions should show up here in realtime...</p>
    <p>ETH balance: {{balance/1e18}} Ether (${{Math.round(balance/1e16*price.USD)/100}})</p>
    <table>
      <thead>
        <tr>
          <th v-for="header in headers" v-text="header"></th>
        </tr>
      </thead>
      <tbody>
        <template v-for="tx in transactionList">
          <tr>
            <td><router-link :to="'/address/' + tx.from">{{tx.from.slice(0,10)}}...</router-link></td>
            <td><router-link :to="'/address/' + tx.to">{{tx.to.slice(0,10)}}...</router-link></td>
            <td><router-link :to="'/tx/' + tx.hash">{{Math.round(tx.value/1e10)/1e8}} Ether</router-link> (${{Math.round(tx.value/1e16*price.USD)/100}})</td>
          </tr>
        </template>
      </tbody>
    </table>
    <div class="comments">
      <VueDisqus shortname="ethereumnetwork" :identifier="$route.path" :url="'https://ethereum.network' + $route.path"></VueDisqus>
    </div>
  </div>
</template>

<script>
import VueQrcode from 'vue-qrcode-component'
import VueDisqus from 'vue-disqus/VueDisqus.vue'
export default {
  name: 'block',
  props: ['searchField'],
  data () {
    return {
      msg: 'the network explorer is in the works...',
      balance: -0,
      headers: ['from', 'to', 'value'],
      transactionList: []
    }
  },
  beforeCreate () {
    fetch('/api/balance/' + this.$route.params.id)
    .then((response) => { return response.json() })
    .then((balance) => {
      this.balance = balance
    })
    fetch('https://api.etherscan.io/api?module=account&action=txlist&sort=desc&address=' + this.$route.params.id)
    .then((response) => { return response.json() })
    .then((transactionList) => {
      this.transactionList = transactionList.result
    })
  },
  mounted () {
    this.$store.state.dsClient.event.subscribe('pending/' + this.$route.params.id, (txData) => {
      this.transactionList.unshift(txData)
      this.transactionList = this.transactionList.slice(0, 50)
    })
  },
  components: {
    VueQrcode,
    VueDisqus
  },
  computed: {
    price: function () {
      return this.$store.state.price
    }
  }
}
</script>

<style>
  .qrcode {
    float:right;
    padding: 1%;
    color: white;
  }
</style>
