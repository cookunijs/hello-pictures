<template>
  <v-form>

    <v-flex xs12 class="">
      <v-text-field
        label="Select Image"
        @click='pickImage'
        v-model='imageName'
        prepend-icon="mdi-camera"/>
      <input
        type="file"
        ref="image"
        accept="image/*"
        style="display: none"
        show-size
        counter
        plain
        @change="onUpload"
      />
    </v-flex>

    <v-card
      max-width="400"
      class="mx-auto"
    >
      <v-card-text  v-if="inputImage">
        <v-subheader>MetaDatas</v-subheader>
        <v-chip-group
          multiple
          column
          active-class="primary--text"
        >
          <v-chip v-for="metaDataKey in metaDataKeys" :key="metaDataKey">
            {{metaDataKey}}：{{ metaData[metaDataKey] }}
          </v-chip>
        </v-chip-group>
      </v-card-text>
    </v-card>

    <v-text-field
      v-model="name"
      label="Name"
      required
    ></v-text-field>
    <v-text-field
      v-model="auther"
      label="Auther"
      required
    ></v-text-field>
    <v-btn class="mr-4" @click="handleOk">submit</v-btn>

  </v-form>
</template>

<script>
import EXIF from 'exif-js'
import ipfs from '../plugins/ipfs';

export default {
  components: {
  },
  data() {
    return {
      inputImage: false,
      image: '',
      name: '',
      auther: '',
      imageName: '',
      imageFile: '',
      imageUrl: '',
      buffer: '',
      imgHash: "",
      metaDataKeys:[
        "Make",
        "Model",
        "DateTimeOriginal",
        "DateTime",
        "PixelXDimension",
        "PixelYDimension",
        "ExposureTime",
        "FNumber",
        "FocalLength",
        "Flash",
        "ISOSpeedRatings",
        "GPSLatitudeRef",
        "GPSLatitude",
        "GPSLongitudeRef",
        "GPSLongitude",
        "GPSDOP"
      ],
      metaData: {
        Make: null,
        Model: null,
        DateTimeOriginal: null,
        DateTime: null,
        PixelXDimension: null,
        PixelYDimension: null,
        ExposureTime: null,
        FNumber: null,
        FocalLength: null,
        Flash: null,
        ISOSpeedRatings: null,
        GPSLatitudeRef: null,
        GPSLatitude: null,
        GPSLongitudeRef: null,
        GPSLongitude: null,
        GPSDOP: null,
      },
    }
  },
  methods: {
    async create(){
      this.$emit("create", {
        name: this.name,
        auther: this.auther,
        metaData: this.metaData,
        metaDataKeys: this.metaDataKeys,
        imgHash: this.imgHash
      })
    },
    pickImage() {
      this.$refs.image.click()
    },
    async onUpload() {
      const files = event.target.files
      const file = files[0]
      await this.generateMetaData(file)
      // await this.captureFile(file)
      const reader = await new FileReader()
      if (typeof file !== 'undefined') {
        await reader.readAsArrayBuffer(file);
        reader.onloadend = async () => {
          this.buffer = await this.convertToBuffer(reader.result)
          console.log(this.buffer)
        }
      } else {
        this.buffer =  ''
      }
      this.inputImage = true
    },
    handleOk() {
      if (!this.buffer) {
        alert('Please fill in the information.');
      } else {
        this.onSubmit()
      }
    },
    generateMetaData(file){
      EXIF.getData(file,()=> {
        console.log(file.exifdata)
        this.metaData.Make = file.exifdata.Make
        this.metaData.Model = file.exifdata.Model
        this.metaData.DateTimeOriginal = file.exifdata.DateTimeOriginal
        this.metaData.DateTime = file.exifdata.DateTime
        this.metaData.PixelXDimension = file.exifdata.PixelXDimension
        this.metaData.PixelYDimension = file.exifdata.PixelYDimension
        this.metaData.ExposureTime = this.fractionCalculation(file.exifdata.ExposureTime)
        this.metaData.FNumber = this.fractionCalculation(file.exifdata.FNumber)
        this.metaData.FocalLength = this.fractionCalculation(file.exifdata.FocalLength)
        this.metaData.Flash = file.exifdata.Flash
        this.metaData.ISOSpeedRatings = file.exifdata.ISOSpeedRatings
        this.metaData.GPSLatitudeRef = file.exifdata.GPSLatitudeRef
        this.metaData.GPSLatitude = this.connectionGPSValue(file.exifdata.GPSLatitude)
        this.metaData.GPSLongitudeRef = file.exifdata.GPSLongitudeRef
        this.metaData.GPSLongitude = this.connectionGPSValue(file.exifdata.GPSLongitude)
        this.metaData.GPSDOP = this.fractionCalculation(file.exifdata.GPSDOP)
        console.log(this.metaData)
      })
    },
    fractionCalculation(data) {
      return data.numerator/data.denominator
    },
    connectionGPSValue(data) {
      let result = ''
      data.forEach(data => {
        if(!result == '') result = result + ','
        result = result + String(this.fractionCalculation(data))
      })
      return result
    },
    async onSubmit() {
      this.$root.loading = true;
      let imgHash;
      await ipfs.add(this.buffer)
        .then((hashedImg) => {
          imgHash = hashedImg[0].hash;
          console.log("imgHash: " + imgHash)
          this.imgHash = imgHash
        })
      this.create()
    },
    async convertToBuffer(reader) {
      return Buffer.from(reader);
    },
  },
  }

</script>