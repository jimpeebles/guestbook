<template>
  <div v-if="isDrizzleInitialized" id="app">
    <h1>Sign the Guestbook</h1>
    <!--Component from the Drizzle plugin to allow filling of smart contract fields-->
    <drizzle-contract-form
      contractName="Guestbook"
      method="signBook"
      :placeholders="['Name']"
    />
    <h2>Guests:</h2>
    <!--Get list of names and transform bytes into Utf8-->
    <ul v-if="getNames">
      <li v-for="(name, i) in getNames" :key="i">{{ utils.toUtf8(name) }}</li>
    </ul>
  </div>
  <div v-else>
    Loading application...
  </div>
</template>

<script>
// Import mapGetters to allow easy access to the getters in Vuex as computed properties
import { mapGetters } from "vuex";

export default {
  name: "app",
  computed: {
    // Use mapGetters to bring in needed methods from the Drizzle plugin
    ...mapGetters("drizzle", ["drizzleInstance", "isDrizzleInitialized"]),
    ...mapGetters("contracts", ["getContractData"]),

    // Subscribe to the getNames method of your smart contract. 
    // In case there's no data yet, return false to prevent display errors.
    getNames() {
      let data = this.getContractData({
        contract: "Guestbook",
        method: "getNames"
      });
      if (data === "loading") return false;
      return data;
    },

    // Utilities needed to transform bytes to strings
    utils() {
      return this.drizzleInstance.web3.utils;
    }
  },

  // Register smart contract before the component mounts to ensure data is available
  created() {
    this.$store.dispatch("drizzle/REGISTER_CONTRACT", {
      contractName: "Guestbook",
      method: "getNames",
      methodArgs: []
    });
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
ul {
  padding: 0px;
}
ul,
li {
  list-style: none;
}
</style>
