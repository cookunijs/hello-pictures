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
import Web3 from 'web3'
const web3 = new Web3('https://rinkeby.infura.io/')

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

      const account = web3.eth.accounts.privateKeyToAccount(localStorage.getItem("privateKey"))
			console.log(1)
			///小数点がサポートされていないここエラー
			const hash = await web3.utils.soliditySha3(
				data.imgHash,
				data.metaData.Make,
        data.metaData.Model,
        data.metaData.DateTimeOriginal,
        data.metaData.DateTime,
        data.metaData.PixelXDimension,
        data.metaData.PixelYDimension,
        data.metaData.ExposureTime,
        data.metaData.FNumber,
        data.metaData.FocalLength,
        data.metaData.Flash,
        data.metaData.ISOSpeedRatings,
        data.metaData.GPSLatitudeRef,
        data.metaData.GPSLatitude,
        data.metaData.GPSLongitudeRef,
        data.metaData.GPSLongitude,
        data.metaData.GPSDOP,
			)
						console.log(hash)

      const sig = await web3.eth.accounts.sign(hash, localStorage.getItem("privateKey"))
      const sendData = {
          address: account.address,
          sig: sig,
          data: data
			}
			console.log(sendData)
      //firebase functions
    },
  }
}
</script>
