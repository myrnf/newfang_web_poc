<template>
  <el-row class="uploaderContainer">
    <el-col :span="12" :offset="6">
      <el-row class="uploadBtnContainer">
        <input type="file" ref="fileSelect" style="display: none" v-on:change="handleFileUpload()">
        <el-button class="uploadBtn" @click="handleUploadBtnClick()">Upload File</el-button>
        <div class="uploadTip">Max File size = 10MB</div>
      </el-row>
      <el-row class="filesContainer">
        <el-col class="noFiles" v-if="noFiles">No Files Uploaded</el-col>
        <el-col class="fileContainer" :span="24" v-for="(file, index) in files" :key="index" v-else>
          <el-card class="file">
            <el-col :span="9" class="fileName">{{ file.name }}</el-col>
            <el-col :span="3" class="fileSize">{{ file.size | sizeFilter }}</el-col>
            <transition name="fade-transform" mode="out-in">
              <el-col :span="12" v-if="file.fuploading"><el-progress style="text-align: left;" :percentage="upPercentage"></el-progress></el-col>
              <el-col :span="12" v-else-if="file.fdownloading"><el-progress style="text-align: left;" :percentage="downPercentage"></el-progress></el-col>
              <el-col :span="2" :offset="10" v-else-if="file.fdeleting"><i class="el-icon-loading"></i></el-col>
              <el-col :span="4" :offset="8" v-else>
                <el-button type="success" plain class="fileCardBtn" icon="el-icon-download" circle @click="handleFileDownload(file.name, file.uri, index)"></el-button>
                <el-button type="danger" plain class="fileCardBtn" icon="el-icon-delete" circle @click="handleFileDelete(file.uri, index, file.convergence)"></el-button>
              </el-col>
            </transition>
          </el-card>
        </el-col>
      </el-row>
    </el-col>
  </el-row>
</template>

<script>
const Uploader = window.newfang_uploader.default
const Downloader = window.newfang_downloader.default
const Utils = window.newfang_utils.default

export default {
  filters: {
    sizeFilter (value) {
      return (value / 1000000).toFixed(2) + 'MB'
    }
  },
  data() {
    return {
      files: [],
      upPercentage: 0,
      downPercentage: 0,
      noFiles: true
    };
  },
  methods: {
    showMsgBox (type, message) {
      this.$message({
        type: type,
        message: message,
        duration: 5000
      })
    },

    handleUploadBtnClick() {
      this.$refs.fileSelect.click()
    },

    handleFileUpload() {
      const file = this.$refs.fileSelect.files[0]
      if (file !== undefined) {
        if (file.size > 100000000) {
          this.showMsgBox('error', 'File size cannot exceed 10MB')
        } else {
          if (this.files.length === 0) {
            this.noFiles = false
          }
          this.files.unshift({ name: file.name, size: file.size, fuploading: true, fdownloading: false, fdeleting: false, uri: '', convergence: '' })

          const convergence = Uploader.generate_convergence()
          const uploader = new Uploader({
            file,
            convergence
          })

          uploader.on("error", (e) => console.log(e));

          uploader.on("upload_progress", (percentage) => {
            this.upPercentage = parseFloat(percentage.toFixed(1))
          })

          uploader.on("upload_complete", (uri) => {
            this.files[0].uri = uri
            this.files[0].convergence = convergence
            this.files[0].fuploading = false
            this.upPercentage = 0
            this.showMsgBox('success', 'Upload Complete!')
            console.log(uri)
          })

          uploader.start_upload()
        }
      } else {
        this.showMsgBox('info', 'File upload aborted')
      }
    },

    handleFileDownload(fileName, uri, index) {
      this.files[index].fdownloading = true
      const downloader = new Downloader(uri, { 
          downloadPath: "",
          type: "Download",
          useWorkers: true
      })

      downloader.on("download_complete", () => {
        console.log("download complete");
        this.files[index].fdownloading = false
        this.downPercentage = 0
        this.showMsgBox('success', 'Download Complete!')
      })
    
      downloader.on("download_progress", (percentage) => {
        console.log("download progress: ", percentage + "%");
        this.downPercentage = parseFloat(percentage.toFixed(1))
      })

      downloader.on("error", (e) => {
        console.log({ e });
        // inform your app
      })

      downloader.start_download(fileName)
    },

    handleFileDelete(uri, index, convergence) {
      this.files[index].fdeleting = true
      const util = new Utils({ uri, convergence })

      util.on("success", () => {
        this.files[index].fdeleting = false
        this.files.splice(index, 1)
        this.showMsgBox('success', 'Delete Complete!')
      })

      util.on("error", error => {
        console.log({ error });
      })

      util.start_delete()
    }
  }
}
</script>

<style>
.uploadBtn {
  border-radius: 20px;
  background: none;
  border: 1px solid #ff7500;
  text-transform: uppercase;
  color: #ff7500;
  padding-left: 32px;
  padding-right: 32px;
  font-weight: 600;
}

.uploadBtn:hover, .uploadBtn:focus {
  background: #ff7500;
  border: 1px solid #ff7500;
  color: #ffffff;
}

.uploadTip {
  font-size: 10px;
  color: #bbb;
  padding: 6px;
}

.filesContainer {
  margin-top: 10px;
}

.fileContainer {
  margin-top: 6px;
}

.fileName {
  padding-bottom: 20px;
  text-align: left;
  font-weight: 600;
}

.fileSize {
  margin-top: 2px;
  font-size: 12px;
  color: #bbb;
}

.fileCardBtn {
  float: right;
  margin-top: -7px !important;
  padding: 8px !important;
  margin-left: 16px;
}
</style>