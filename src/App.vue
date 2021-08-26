<template>
  <div id="app">
    <div class="uploader">
      <avatar-uploader
        :height="150"
        @confirm="confirm"
        :distProps="{ backgroundColor: 'green', format: 'png', distSize: [200, 200] }"
        :maskProps="{ maskBorderRadius: '10px' }"
      >
      </avatar-uploader>
    </div>
    <img :src="distUrl" />
  </div>
</template>

<script>
import AvatarUploader from './components/AvatarUploader.vue';

export default {
  name: 'App',
  components: {
    AvatarUploader
  },
  data() {
    return {
      distUrl: ''
    };
  },
  methods: {
    confirm(url) {
      this.distUrl = url;
    }
  }
};
</script>

<style>
.uploader {
  width: 400px;
  border: 1px solid #aaa;
}
</style>
