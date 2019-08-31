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
import Form from '~/components/Form.vue'
// import VuetifyLogo from '~/components/VuetifyLogo.vue'

export default {
  components: {
    Form
  }
}
</script>
