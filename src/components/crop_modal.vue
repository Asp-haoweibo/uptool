<template>
  <div class="modal is-active" style="text-align:center;">
    <div class="modal-background"></div>
    <div class="modal-card" style="display: inline-block; width: auto;">
      <header class="modal-card-head">
        <p class="modal-card-title">设置封面</p>
        <button v-on:click="$emit('close-modal')" class="delete" aria-label="close"></button>
      </header>
      <section class="modal-card-body">
        <!-- Content ... -->
        <input id="cover-input" v-on:change="load" type='file'>
        <img id="cropper" :src="image_base64">
        <div class="level box" v-if="videos && videos.length != 0">
          <div class="level-item" v-if="preCroppedCovers.length==0">
            <!-- no precropped images, display a loading badage-->
            <font-awesome-icon icon="spinner" />
          </div>
          <div v-for="img in preCroppedCovers" class="level-item">
            <img :src="img" class="image is-64x64" @click="setImage(img)" />
          </div>
        </div>
      </section>
      <footer class="modal-card-foot">
        <button v-on:click="save" class="button is-success">保存</button>
        <button v-on:click="reset" class="button">重置</button>
      </footer>
    </div>
  </div>
</template>
<script>
import Cropper from "cropperjs";
import "cropperjs/dist/cropper.css";
import { resolve } from "path";

export default {
  name: "crop-modal",
  methods: {
    load: function(ev) {
      const files = ev.target.files;

      if (files && files[0]) {
        const reader = new FileReader();

        reader.onload = e => {
          this.image_base64 = e.target.result;
          // var image = document.getElementById("cropper");
        };
        reader.readAsDataURL(files[0]);

        reader.onloadend = () => {
          this.cropper.replace(this.image_base64);
        };

        this.initialize_cropper();
      }
    },
    save: function() {
      this.$store.commit("setCover", this.image_base64);
      this.$emit("close-modal");
      if (this.cropper.getCroppedCanvas()) {
        this.$store.commit(
          "setCroppedCover",
          this.cropper.getCroppedCanvas().toDataURL()
        );
        this.$store.commit("setCropperData", this.cropper.getData());
        console.log(this.$store.state);
      } else {
        this.$store.commit("setCroppedCover", "");
        this.$store.commit("setCropperData", null);
      }
    },
    reset: function() {
      this.image_base64 = "";
      document.getElementById("cover-input").value = "";
      this.cropper.destroy();
    },
    initialize_cropper: function() {
      var image = document.getElementById("cropper");
      this.cropper = new Cropper(image, {
        viewMode: 0,
        aspectRatio: 16 / 10
      });
    },
    setImage(img) {
      this.image_base64 = img;
      this.cropper.replace(this.image_base64);
    }
  },
  props: ["videos"],
  data() {
    return {
      cropper: null
    };
  },
  computed: {
    image_base64: {
      get() {
        return this.$store.state.cover;
      },
      set(data) {
        this.$store.commit("setCover", data);
      }
    },
    cropper_data: {
      get() {
        return this.$store.state.cropperData;
      },
      set(data) {
        this.$store.commit("setCropperData", data);
      }
    },
    preCroppedCovers() {
      return this.$store.state.preCroppedCovers;
    }
  },
  mounted: function() {
    var image = document.getElementById("cropper");
    this.cropper = new Cropper(image, {
      viewMode: 0,
      aspectRatio: 16 / 10,
      data: this.cropper_data
    });
  }
};
</script>

<style>
#cropper {
  max-width: 100%;
  max-height: calc(100vh - 600px);
}
</style>