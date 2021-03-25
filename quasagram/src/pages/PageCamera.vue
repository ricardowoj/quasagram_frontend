<template>
  <q-page class="constrain-camera q-pa-md">
    <!-- IMAGEM CAMERA E FOOT -->
    <div class="camera-frame q-pa-md">
      <video
        multiple
        auto-expand
        v-show="!imageCaptured"
        ref="video"
        class="full-width"
        autoplay
      ></video>
      <canvas
        v-show="imageCaptured"
        ref="canvas"
        class="full-width"
        height="240"
      />
    </div>
    <div class="text-center q-pa-md">
      <!-- BOTAO FOTO -->
      <q-btn
        v-if="hasCameraSupport"
        @click="captureImage"
        color="grey-10"
        icon="eva-camera"
        size="md"
        round
      />
      <!-- BOTAO UPLOAD FOTO -->
      <q-file
        v-else
        v-model="imageUpload"
        @input="captureImageFallback"
        accept="image/*"
        label="Foto"
      >
        <template v-slot:prepend>
          <q-icon name="eva-attach-2-outline" />
        </template>
      </q-file>
    </div>
    <!-- CAMPO COMENTARIO -->
    <div class="row justify-center q-ma-md">
      <q-input
        v-model="post.caption"
        label="Comentário"
        class="col col-sm-6"
        dense
      />
    </div>
    <!-- CAMPO LOCALIZACAO -->
    <div class="row justify-center q-ma-md">
      <q-input
        v-model="post.location"
        :loading="loadingPosition"
        label="Localização"
        class="col col-sm-6"
        dense
      >
        <template v-slot:append>
          <q-icon
            v-if="!loadingPosition"
            @click="getLocation"
            name="eva-navigation-outline"
          />
        </template>
      </q-input>
    </div>
    <!-- BOTAO POSTAR -->
    <div class="row justify-center q-mt-lg">
      <q-btn
        @click="addPost()"
        color="primary"
        label="Postar"
        unelevated
        rounded
      />
    </div>
  </q-page>
</template>

<script>
import { uid } from "quasar";
require("md-gum-polyfill");

export default {
  name: "PageCamera",
  data() {
    return {
      post: {
        //caption: "",
        //date: Date.now(),
        //location: "",
        file: null
      },
      image: {
        photo: null
      },
      nameImage: uid(),
      imageCaptured: false,
      imageUpload: [],
      hasCameraSupport: true,
      loadingPosition: false
    };
  },
  methods: {
    initCamera() {
      navigator.mediaDevices
        .getUserMedia({
          video: true
        })
        .then(MediaStream => {
          this.$refs.video.srcObject = MediaStream;
          this.$refs.video.play();
        })
        .catch(error => {
          this.hasCameraSupport = false;
          console.error("getUserMedia() error:", error);
        });
    },
    captureImage() {
      let video = this.$refs.video;
      let canvas = this.$refs.canvas;
      canvas.width = video.getBoundingClientRect().width;
      canvas.height = video.getBoundingClientRect().height;
      let context = canvas.getContext("2d");
      context.drawImage(video, 0, 0, canvas.width, canvas.height);
      this.imageCaptured = true;
      this.disableCamera();

      const blobToImage = blob => {
        return new Promise(resolve => {
          const url = this.dataURItoBlob(canvas.toDataURL());
          let img = new Image();
          img.onload = () => {
            URL.revokeObjectURL(url);
            resolve(img);
          };
          img.src = url;
        });
      };

      this.post.file = blobToImage;
      console.log(this.post.file);
    },
    captureImageFallback(file) {
      this.post.photo = file;

      let canvas = this.$refs.canvas;
      let context = canvas.getContext("2d");

      var reader = new FileReader();
      reader.onload = event => {
        var img = new Image();
        img.onload = () => {
          canvas.width = img.width;
          canvas.height = img.height;
          context.drawImage(img, 0, 0);
          this.imageCaptured = true;
        };
        img.src = event.target.result;
      };
      reader.readAsDataURL(file);
    },
    disableCamera() {
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        track.stop();
      });
    },
    dataURItoBlob(dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(",")[1]);

      // separate out the mime component
      var mimeString = dataURI
        .split(",")[0]
        .split(":")[1]
        .split(";")[0];

      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length);

      // create a view into the buffer
      var ia = new Uint8Array(ab);

      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }

      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], { type: mimeString });
      return blob;
    },
    getLocation() {
      this.loadingPosition = true;
      try {
        navigator.geolocation.getCurrentPosition(position => {
          setTimeout(() => {
            let latitude = position.coords.latitude;
            let longitude = position.coords.longitude;
            this.getCityAndCountry(latitude, longitude);
          }, 1000);
        });
      } catch (error) {
        console.log("Erro: ", error);
        this.locationError();
      }
    },
    getCityAndCountry(latitude, longitude) {
      let apiUrl = `https://geocode.xyz/${latitude},${longitude}?json=1`;
      this.$axios
        .get(apiUrl)
        .then(result => {
          this.post.location = `${result.data.city}, ${result.data.country}`;
          this.loadingPosition = false;
        })
        .catch(err => {
          this.locationError();
          this.loadingPosition = false;
        });
    },
    locationError() {
      this.$q.dialog({
        title: "Erro",
        message: "Não foi possível acessar sua localização"
      });
    },
    addPost() {
      const formData = new FormData();
      //caption: this.post.caption,
      //date: this.post.date,
      //location: this.post.location
      formData.append("file", this.post.photo);
      this.$axios
        .post(`http://localhost:8080/images`, formData.photo)
        .then(response => {
          console.log("response", response);
        })
        .catch(err => {
          console.log("err: ", err);
        });
    }
  },
  mounted() {
    this.initCamera();
  },
  beforeDestroy() {
    if (this.hasCameraSupport) {
      this.disableCamera();
    }
  }
};
</script>

<style lang="scss">
.camera-frame {
  border: 2px solid $grey-10;
  border-radius: 10px;
}
</style>
