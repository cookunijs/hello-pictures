<template>
  <v-app>
    <v-card
    class="mx-auto overflow-hidden"
    height="100%"
    width="100%"
    >
      <v-app-bar
        color="accent-4"
        dark
        prominent
      >
        <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>

        <v-toolbar-title>Hello Pictures</v-toolbar-title>

        <div class="flex-grow-1"></div>

        <v-btn icon>
          <v-icon>mdi-magnify</v-icon>
        </v-btn>

        <v-btn icon>
          <v-icon>mdi-filter</v-icon>
        </v-btn>

        <v-btn icon>
          <v-icon>mdi-dots-vertical</v-icon>
        </v-btn>
      </v-app-bar>

      <v-navigation-drawer
        v-model="drawer"
        absolute
        bottom
        temporary
      >
        <v-list
          nav
          dense
        >
          <v-list-item-group
            v-model="group"
            active-class="deep-purple--text text--accent-4"
          >
            <v-list-item>
              <v-list-item-title>Foo</v-list-item-title>
            </v-list-item>

            <v-list-item>
              <v-list-item-title>Bar</v-list-item-title>
            </v-list-item>

            <v-list-item>
              <v-list-item-title>Fizz</v-list-item-title>
            </v-list-item>

            <v-list-item>
              <v-list-item-title>Buzz</v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-navigation-drawer>
      <v-card-text>
        <nuxt />
      </v-card-text>
    </v-card>
  </v-app>
</template>

<script>
import Web3 from 'web3'
const web3 = new Web3('https://rinkeby.infura.io/')

export default {
  data () {
    return {
      drawer: false,
      group: null,
    }
  },
  mounted: async function() {
    if(!localStorage.getItem("wallet")) {
      const result = await web3.eth.accounts.create()
      await localStorage.setItem("privateKey", result.privateKey)
    }
  },
  watch: {
    group () {
      this.drawer = false
    },
  },
}
</script>
