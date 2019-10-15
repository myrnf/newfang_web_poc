<template>
  <div class="container">
    <Navbar />
    <div class="headerText">Newfang Web SDK Test App</div>
    <Upload />
  </div>
</template>

<script>
import Navbar from '~/components/Navbar.vue'
import Upload from '~/components/Upload.vue'

export default {
  components: {
    Navbar,
    Upload
  },
  data() {
    return {
      fileList: []
    };
  },
  methods: {
    
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  text-align: center;
}

.headerText {
  font-weight: 200;
  font-size: 60px;
  margin-top: 20px;
  margin-bottom: 24px;
  color: #ddd;
}
</style>
