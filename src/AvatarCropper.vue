<template>
  <div class="avatar-cropper" v-if="dataUrl">
    <div class="avatar-cropper-mark"><a href="javascript:;" class="avatar-cropper-close" @click="destroy">&times;</a></div>
    <div class="avatar-cropper-container">
      <div class="avatar-cropper-image-container">
        <img :src="dataUrl" alt="" @load.stop="createCropper" />
      </div>
      <div class="avatar-cropper-footer">
        <button class="avatar-cropper-btn" @click="destroy" v-text="labels.cancel">Cancel</button>
        <button class="avatar-cropper-btn" @click="submit" v-text="labels.submit">Submit</button>
      </div>
    </div>
  </div>
</template>

<script>
  import 'cropperjs/dist/cropper.css'
  import Cropper from 'cropperjs'
  import axios from 'axios'
  import VueAxios from 'vue-axios'

  Vue.use(VueAxios, axios)


  export default {
    props: {
      trigger: {
        type: [String, Element],
        required: true
      },
      uploadHandler: {
        type: Function,
      },
      aspectRatio: {
        type: String,
      },
      uploadUrl: {
        type: String,
      },
      uploadHeaders: {
        type: Object,
      },
      uploadFormName: {
        type: String,
        default: 'file'
      },
      uploadFormData: {
        type: Object,
        default() {
          return {}
        }
      },
      mimes: {
        type: String,
        default: 'image/png, image/gif, image/jpeg, image/bmp, image/x-icon'
      },
      labels: {
        submit: "提交",
        cancel: "取消"
      }
    },
    data() {
      return {
        cropper: undefined,
        dataUrl: undefined
      }
    },
    methods: {
      destroy() {
        this.cropper.destroy()
        this.dataUrl = undefined
      },
      submit() {
        if (this.uploadUrl) {
          this.uploadImage()
        } else if (this.uploadHandler) {
          this.uploadHandler(this.cropper)
        } else {
          throw new Error('No upload handler found.')
        }
        this.destroy()
      },
      pickImage() {
        let id = 'avatar-img-input'
        let fileInput = document.querySelector('input#'+id+'[type=file]')
        if (fileInput == null) {
          fileInput = document.createElement('input')
          fileInput.id = id
          fileInput.setAttribute('type', 'file')
          fileInput.setAttribute('accept', this.mimes)
          fileInput.addEventListener('change', () => {
            if (fileInput.files != null && fileInput.files[0] != null) {
              let reader = new FileReader()
              reader.onload = (e) => {
                this.dataUrl = e.target.result
              }
              reader.readAsDataURL(fileInput.files[0])
            }
          })
        }
        fileInput.click()
      },
      createCropper() {
        let image = document.querySelector('.avatar-cropper-image-container img')
        this.cropper = new Cropper(image, {
          aspectRatio: Number(this.aspectRatio),
          autoCropArea: 1,
          viewMode: 1,
          movable: false,
          zoomable: false,
        })
      },
      uploadImage() {
        this.cropper.getCroppedCanvas().toBlob((blob) => {
          let form = new FormData()
          let data = Object.assign({}, this.uploadFormData)

          data[this.uploadFormName] = blob

          for (let key in data) {
            form.append(key, data[key])
          }

          this.axios.post(this.uploadUrl, form).then((req) => {
            this.$emit('uploaded', req)
          }).catch(err => {
            console.error('Error: ', err)
          })

        })
      }
    },
    mounted() {
      let trigger = typeof this.trigger == 'object' ? this.trigger : document.querySelector(this.trigger)
      if (!trigger) {
        throw new Error('No avatar make trigger found.')
      }
      trigger.addEventListener('click', this.pickImage)
    }
  }
</script>

<style lang="scss">
  .avatar-cropper {
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    position: fixed;
    top:0;
    left:0;
    right:0;
    bottom:0;
    z-index: 99999;

    .avatar-cropper-close {
      float: right;
      padding: 20px;
      font-size: 3rem;
      color: #fff;
      font-weight: 100;
      text-shadow: 0px 1px rgba(40, 40, 40,.3);
    }

    .avatar-cropper-mark {
      position: fixed;
      top:0;
      left:0;
      right:0;
      bottom:0;
      background: rgba(0, 0, 0, .10);
    }
    .avatar-cropper-container {
      background: #fff;
      z-index: 999;
      box-shadow: 1px 1px 5px rgba(100, 100, 100, .14);

      .avatar-cropper-image-container {
        position: relative;
        max-width: 400px;
        height: 300px;
      }
      img {
        max-width: 100%;
        height: 100%;
      }

      .avatar-cropper-footer {
        display: flex;
        align-items: stretch;
        align-content: stretch;
        justify-content: space-between;

        .avatar-cropper-btn {
          width: 50%;
          padding: 15px 0;
          cursor: pointer;
          border: none;
          background: transparent;
          outline: none;
          &:hover {
            background-color: #2aabd2;
            color: #fff;
          }
        }
      }
    }
  }
</style>
