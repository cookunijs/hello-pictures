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
			// if(files[0] !== undefined) {
			// 	this.imageName = files[0].name
			// 	if(this.imageName.lastIndexOf('.') <= 0) {
			// 		return
			// 	}
			// 	const fr = new FileReader ()
			// 	fr.readAsDataURL(files[0])
			// 	fr.addEventListener('load', () => {
			// 		this.imageUrl = fr.result
			// 		this.imageFile = files[0]
			// 	})
			// } else {
			// 	this.imageName = ''
			// 	this.imageFile = ''
			// 	this.imageUrl = ''
      // }
      this.inputImage = true
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
    create(){
      this.$emit("create", {
        imageName: this.imageName,
        imageFile: this.imageFile,
        imageUrl: this.imageUrl,
        name: this.name,
        auther: this.auther,
        metaData: this.metaData,
        metaDataKeys: this.metaDataKeys,
      })
    },
    // async captureFile(file) {
    //  const reader = await new FileReader()
    //  console.log(file)
    //   if (typeof file !== 'undefined') {
    //     await reader.readAsArrayBuffer(file);
    //     reader.onloadend = async () => {
    //       console.log(await this.convertToBuffer(reader.result))
    //       this.buffer = await this.convertToBuffer(reader.result)
    //     }
    //   } else {
    //     this.buffer =  ''
    //   }
    // },
    async convertToBuffer(reader) {
      return Buffer.from(reader);
    },
    onSubmit() {
      console.log(this.buffer)
      this.$root.loading = true;
      let imgHash;
      ipfs.add(this.buffer)
        .then((hashedImg) => {
          imgHash = hashedImg[0].hash;
          console.log("imgHash: " + imgHash)
          this.imgHash = imgHash
        })
    },
    handleOk() {
      if (!this.buffer) {
        alert('Please fill in the information.');
      } else {
        this.onSubmit()
      }
    },
  },
  }

</script>