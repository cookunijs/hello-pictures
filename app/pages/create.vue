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
        <Form
          key="create"
          @create="create"
        />
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
import Form from '~/components/Form.vue'

export default {
  components: {
    Form
  },
  data() {
    return {
      step: 0,
      memos: [],
    }
  },
  methods: {
    async create(data) {
      console.log(data)
    },
  }
}
</script>
