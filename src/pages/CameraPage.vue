<template>
  <!-- page width -->
  <q-page class="constrain-more q-pa-md">
    <!-- image width/ div -->
    <div class="cameraframe q-pa-md">
      <!-- ref allows us to access the return of the stream in our js data -->
      <!-- show video element-->
      <video v-show="!imageCaptured" ref="video" class="full-width" autoplay />
      <!-- canvas element to capture the image -->
      <!-- don't show video element-->
      <canvas
        v-show="imageCaptured"
        ref="canvas"
        class="full-width"
        height="240"
      />
    </div>

    <div class="text-center q-pa-md">
      <!-- camera button -->
      <!-- v-if has camerasupport the button will appear if not it will disappear -->
      <q-btn
        v-if="hasCameraSupport"
        @click="captureImage"
        round
        color="pink-5"
        icon="fas fa-camera"
        size="18px"
      />
      <!-- this creates an attach file allowing users to pick a picture from their own pc -->
      <q-file
        v-else
        v-model="imageUpload"
        @input="captureImageFallback"
        label="Choose an image"
        accept="image/*"
        outlined
      >
        <template v-slot:prepend>
          <q-icon name="fas fa-paperclip" />
        </template>
      </q-file>
      <!-- text row -->
      <div class="row justify-center q-ma-md">
        <!-- first text -->
        <q-input
          v-model="post.caption"
          class="col col-sm-6"
          label="Caption"
          dense
        />
      </div>
      <!-- text row -->
      <div class="row justify-center q-ma-md">
        <!-- second text -->
        <!-- location input and button -->
        <q-input
          v-model="post.location"
          :loading="locationLoading"
          class="col col-sm-6"
          label="Location"
          dense
        >
          <!-- location icon -->
          <template v-slot:append>
            <q-btn
              v-if="!locationLoading && locationSupported"
              @click="getLocation"
              icon="fas fa-globe-europe fa-2x"
              dense
              flat
            />
          </template>
        </q-input>
      </div>
      <!-- button row -->
      <div class="row justify-center q-mt-lg">
        <!-- post image button -->
        <q-btn outline rounded color="pink-5" label="Post image" />
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'

import { uid } from 'quasar'

export default defineComponent({
  name: 'CameraPage',
  data() {
    return {
      // creating a post information
      post: {
        id: uid(),
        caption: '',
        location: '',
        photo: null,
        date: Date.now()
      },
      imageCaptured: false, //only capture the image by the captured image
      imageUpload: [],
      hasCameraSupport: true, // if the user has camera support the button will appear
      locationLoading: false
    }
  },
  computed: {
    // if users browser doesn't have geolocation the location button will be hidden
    locationSupported() {
      if ('geolocation' in navigator) return true
      return false
    }
  },
  methods: {
    initCamera() {
      //requesting users camera
      navigator.mediaDevices
        .getUserMedia({
          //getting users camera request
          video: true //this will return a promise with allowing of a camera
        })
        .then(stream => {
          this.$refs.video.srcObject = stream
        })
        .catch(error => {
          //if the user doesn't have camera support this will avoid the init camera to bug out
          this.hasCameraSupport = false
        })
    },
    captureImage() {
      let video = this.$refs.video //access to the video
      let canvas = this.$refs.canvas //access to the canvas
      canvas.width = video.getBoundingClientRect().width //making canvas width the same width as the video
      canvas.height = video.getBoundingClientRect().height //making canvas height the same height as the video
      let context = canvas.getContext('2d') //allows us to draw in 2d
      context.drawImage(video, 0, 0, canvas.width, canvas.height) // draw image top left corner, y coordinate, and width and height
      this.imageCaptured = true //after image has been selected it's now captured
      this.post.photo = this.dataURItoBlob(canvas.toDataURL()) // after capture the image save it to photo array object and save it as a url data base 64
      this.disableCamera() //calls the disable camera method after the image has been taken
    },
    captureImageFallback(file) {
      //return the file that the user has chosen
      this.post.photo = file

      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')

      let reader = new FileReader() //upload image file to canvas
      reader.onload = event => {
        let img = new Image()
        img.onload = () => {
          canvas.width = img.width
          canvas.height = img.height
          context.drawImage(img, 0, 0) //draw image to the canvas
          this.imageCaptured = true
        }
        img.src = event.target.result
      }
      reader.readAsDataURL(file.target.files[0]) // this is where it reads the file that has been selected and I was able to fix yesssss!!!
    },
    disableCamera() {
      //disable video after a picture has been taken
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        // for one track taken
        track.stop() // the track then stops
      })
    },
    //bob is an image file where we can store our images in a firebase storage
    dataURItoBlob(dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(',')[1])

      // separate out the mime component
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]

      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length)

      // create a view into the buffer
      var ia = new Uint8Array(ab)

      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i)
      }

      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], {
        type: mimeString
      })
      return blob
    },
    getLocation() {
      //get users location from the location globe button
      this.locationLoading = true
      navigator.geolocation.getCurrentPosition(
        position => {
          this.getCityAndCountry(position)
        },
        err => {
          this.locationError()
        },
        { timeout: 7000 }
      )
    },
    getCityAndCountry(position) {
      let apiUrl = `https://geocode.xyz/${position.coords.latitude}, ${position.coords.longitude}?json=1`
      this.$axios
        .get(apiUrl)
        .then(result => {
          this.locationSuccess(result)
        })
        .catch(err => {
          this.locationError()
        })
    },
    locationSuccess(result) {
      // if the location is success then add city and country
      this.post.location = result.data.city
      if (result.data.country) {
        // if the country is located then add country to data
        this.post.location += `, ${result.data.country}`
      }
      this.locationLoading = false
    },
    locationError() {
      //location if not found this error dialog will show
      this.$q.dialog({
        title: 'Error',
        message: 'No location found'
      })
      this.locationLoading = false
    }
  },
  mounted() {
    this.initCamera()
  },
  beforeUnmount() {
    //disable camer after user leaves the camerapage, check this part once u get home
    if (this.hasCameraSupport) {
      this.disableCamera()
    }
  }
})
</script>

<style lang="scss">
.cameraframe {
  border: 2px solid $pink-5;
  border-radius: 15px;
}
</style>
