<template>
  <div class="home">
    <div id="tagline">
      a simple tool for creators to see secondary sales from
      <a href="https://www.fxhash.xyz/" target="_blank">FXHASH</a>
      <input v-model="userAddress" placeholder="Wallet address" />
      <button v-on:click="getObjktNames">go</button>
    </div>
    <table v-if="addressEntered && objktIds.length > 0">
      <tr>
        <td></td>
        <td><h3>Token name</h3></td>
        <td><h3>Buyer</h3></td>
        <td><h3>Seller</h3></td>
        <td><h3>Sold for</h3></td>
        <td>
          <h3>Royalties<br />Received</h3>
        </td>
        <td><h3>Timestamp</h3></td>
      </tr>
      <tr v-for="(obj, i) in objktNames" :key="i">
        <td>{{i+1}}</td>
        <td>
          <a
            :href="'https://www.fxhash.xyz/objkt/' + objktIds[i]"
            target="_blank"
            >{{ obj }}</a
          >
        </td>
        <td>
          <a :href="'https://tzkt.io/' + buyerAddresses[i]" target="_blank">{{
            shortenAddress(buyerAddresses[i])
          }}</a>
        </td>
        <td>
          <a :href="'https://tzkt.io/' + sellerAddresses[i]" target="_blank">
            {{ shortenAddress(sellerAddresses[i]) }}</a
          >
        </td>
        <td>{{ soldPrice[i].toFixed(3) }}</td>
        <td>{{ royaltyAmt[i].toFixed(3) }}</td>
        <td>
          <a :href="'https://tzkt.io/' + opHashes[i]" target="_blank">{{
            timeStamps[i]
          }}</a>
        </td>
      </tr>
      <tr>
        <td></td>
        <td></td>
        <td></td>
        <td>Total:</td>
        <td>{{soldTotal.toFixed(3)}}</td>
        <td>{{ royaltyTotal.toFixed(3) }}</td>
        <td></td>
      </tr>
    </table>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
  data: () => ({
    userAddress: "",
    tzkt: "https://api.tzkt.io/v1/",
    sender: "KT1Xo5B7PNBAeynZPmca4bRh6LQow4og1Zb9",
    tokenContract: "KT1KEa8z6vWXDJrVqtMrAeDVzsvxat3kHaCE",
    bcd: "https://api.better-call.dev/v1/",
    query: "",
    opHashes: [],
    objktIds: [],
    soldPrice: [],
    royaltyAmt: [],
    objktNames: [],
    buyerAddresses: [],
    sellerAddresses: [],
    timeStamps: [],
    addressEntered: false,
  }),
  created() {
    let path = this.$router.history.current.fullPath;
    if (path.length > 2) {
      this.calledFromHistory = true;
      this.userAddress = path.substring(7);
      this.getObjktNames();
    }
  },
  computed: {
    royaltyTotal: function(){
        return this.sumArray(this.royaltyAmt)
    },
    soldTotal: function(){
      return this.sumArray(this.soldPrice)
    }
  },
  methods: {
    sumArray: function (arr) {
      let temp = arr.map((f) => parseFloat(f));
      return temp.reduce((prev, curr) => prev + curr);
    },
    clearData: function () {
      this.opHashes = [];
      this.objkIds = [];
      this.soldPrice = [];
      this.royaltyAmt = [];
      this.objktNames = [];
      this.buyerAddresses = [];
      this.sellerAddresses = [];
    },
    shortenAddress: function (addr) {
      let first = addr.substring(0, 4);
      let last = addr.substring(31);
      return first + "..." + last;
    },
    getHashes: async function () {
      const response = await fetch(this.tzkt + this.query);
      const data = await response.json();
      data.forEach((d) => {
        //make sure transaction is not user buying
        if (d.initiator.address != this.userAddress) {
          this.opHashes.push(d.hash);
          this.timeStamps.push(d.timestamp);
        }
      });
    },
    getTransactionData: async function () {
      await this.getHashes();
      for (let i = 0; i < this.opHashes.length; i++) {
        const response = await fetch(
          this.tzkt + "operations/" + this.opHashes[i]
        );
        const data = await response.json();
        let id = data[0].diffs ? 0 : 1; //check if data has diffs
        let content = data[id].diffs[0].content.value;
        this.objktIds.push(content.objkt_id);
        this.soldPrice.push(content.price * 0.000001);
        this.royaltyAmt.push(
          (this.soldPrice[i] * content.royalties * 0.001)
        );
        this.buyerAddresses.push(data[id].sender.address);
        this.sellerAddresses.push(data[data.length - 1].target.address);
      }
    },
    getObjktNames: async function () {
      this.clearData();
      this.query = `accounts/${this.userAddress}/operations?sender=${this.sender}&type=transaction&limit=1000`;
      this.$router.replace({
        query: {
          ...this.$router.query,
          addr: this.userAddress,
        },
      });
      this.addressEntered = true;
      await this.getTransactionData();
      for (let i = 0; i < this.objktIds.length; i++) {
        const response = await fetch(
          this.bcd +
            "tokens/mainnet/metadata?contract=" +
            this.tokenContract +
            "&token_id=" +
            this.objktIds[i]
        );
        const data = await response.json();
        const ipfsid =
          data[0].token_info[Object.keys(data[0].token_info)[0]].substring(7);
        const ipfslink = "https://gateway.ipfs.io/ipfs/" + ipfsid;
        const ipfsResponse = await fetch(ipfslink);
        const ipfsData = await ipfsResponse.json();
        this.objktNames.push(ipfsData.name);
      }
    },
  },
};
</script>

<style>
h3 {
  padding-top: 3vh;
  margin: 0;
}
td {
  padding-right: 2vw;
  text-align:center;
}

tr:nth-child(even) {
  background: #ccc;
}
table {
  padding: 0vh 2vw;
}
#tagline {
  padding: 0 2vw;
}
input {
  margin-left: 1vw;
}
</style>