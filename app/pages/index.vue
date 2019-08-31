<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-flex
      xs12
      sm8
      md6
    >
      <div class="text-center">
        <Form/>
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
// import config from '../config.json'
// import client from '~/plugins/ethereum-client'
// import functions from '~/plugins/functions'
// import axios from 'axios'
import Form from '~/components/Form.vue'

export default {
  components: {
    Form
  },
}
</script>
