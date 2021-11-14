<template>
  <div class="home">
    <div>
      a simple tool to show secondary sales from
      <a href="https://www.fxhash.xyz/" target="_blank">FXHASH</a>
    </div>
    <input v-model="userAddress" placeholder="Wallet address" />
    <button v-on:click="getObjktNames">go</button>
    <table v-if="addressEntered && objktIds.length > 0">
      <tr>
        <td>Token name</td>
        <td>Buyer</td>
        <td>Seller</td>
        <td>Sold for</td>
        <td>Royalties Received</td>
        <td>Timestamp</td>
      </tr>
      <tr v-for="(obj, i) in objktNames" :key="i">
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
        <td>{{ soldPrice[i].toFixed(2) }}</td>
        <td>{{ royaltyAmt[i] }}</td>
        <td>
          <a :href="'https://tzkt.io/' + opHashes[i]" target="_blank">{{
            timeStamps[i]
          }}</a>
        </td>
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
    calledFromHistory: false,
  }),
  created() {
    let path = this.$router.history.current.fullPath;
    if (path.length > 2) {
      this.calledFromHistory = true;
      this.userAddress = path.substring(7);
      this.getObjktNames();
    }
  },
  methods: {
    shortenAddress: function (addr) {
      let first = addr.substring(0, 4);
      let last = addr.substring(31);
      return first + "..." + last;
    },
    getHashes: async function () {
      const response = await fetch(this.tzkt + this.query);
      const data = await response.json();
      data.forEach((d) => {
        this.opHashes.push(d.hash);
        this.timeStamps.push(d.timestamp);
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
          (this.soldPrice[i] * content.royalties * 0.001).toFixed(2)
        );
        this.buyerAddresses.push(data[id].sender.address);
        this.sellerAddresses.push(data[data.length - 1].target.address);
      }
    },
    getObjktNames: async function () {
      this.query = `accounts/${this.userAddress}/operations?sender=${this.sender}&type=transaction`;
      if (!this.calledFromHistory) {
        this.$router.replace({
          query: {
            ...this.$router.query,
            addr: this.userAddress,
          },
        });
      }
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
td {
  padding: 0 20px;
}
</style>