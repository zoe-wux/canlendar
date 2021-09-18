<template>
  <div>
    <a @click="openModal">修改头像</a>
    <a-modal
      title="修改头像"
      :visible="isVisible"
      centered
      @ok="saveModal"
      @cancel="cancelModal"
      okText="保存"
      cancelText="取消"
    >
      <div class="container">
        <div style="margin-bottom: 20px;">
          <a-button type="primary" style="position: relative;">
            上传图片
            <input
              type="file"
              class="upload_file"
              accept="image/png, image/jpeg, image/gif, image/jpg"
              @change="uploadImg"
              ref="uploadImg"
            />
          </a-button>
        </div>
        <!-- 裁剪区 -->
        <div class="crop_area">
          <vueCropper
            ref="cropper"
            :img="option.img"
            :outputSize="option.size"
            :outputType="option.outputType"
            :info="true"
            :full="option.full"
            :canMove="option.canMove"
            :canMoveBox="option.canMoveBox"
            :fixedBox="option.fixedBox"
            :original="option.original"
            :autoCrop="option.autoCrop"
            :autoCropWidth="option.autoCropWidth"
            :autoCropHeight="option.autoCropHeight"
            :high="option.high"
            :maxImgSize="option.maxImgSize"
            :mode="option.mode"
            :limitMinSize="option.limitMinSize"
            @realTime="realTime"
            @cropMoving="cropMoving"
          ></vueCropper>
        </div>
        <div class="preview">
          预览效果
          <div class="preview__img">
            <img :src="previews.url" :style="previews.img" />
          </div>
          80*80
          <div class="preview__img" style="zoom: 0.5;">
            <img :src="previews.url" :style="previews.img" />
          </div>
          40*40
        </div>
      </div>
    </a-modal>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isVisible: false,
      // 图片更改合集
      previews: {},
      option: {
        img: '', // 裁剪图片的地址
        size: 1,
        outputType: 'png', // 裁剪生成的图片格式
        full: false, // 是否输出原图比例截图
        canMove: true, // 能否拖动图片
        canMoveBox: false, // 能否拖动截图框
        fixedBox: true, // 截图框固定大小
        original: false, // 上传图片时是否显示原始宽高（针对大图 可以铺满）
        autoCrop: true, // 是否自动生成截图框
        // 只有自动截图开启 宽度高度才生效
        autoCropWidth: 80,
        autoCropHeight: 80, // 自动生成截图框宽高
        high: false, // 根据dpr生成适合屏幕的高清图片
        maxImgSize: 3000, // 上传图片时最大大小（默认会压缩尺寸到这个大小）
        mode: '100% auto', // 默认图片渲染方式
        limitMinSize: [100, 120], // 截图框的最小限制
        cropData: {}
      }
    }
  },
  methods: {
    openModal() {
      this.isVisible = true
    },
    cancelModal() {
      this.option.img = null
      this.isVisible = false
    },
    saveModal() {
      this.$refs.cropper.getCropBlob(data => {
        const file = new File([data], 'filename', { type: 'image/png' })
        const formData = new FormData()
        formData.append('file', file)
        this.$axios({
          url: '/szxyDesktop/user/portrait/save',
          method: 'post',
          data: formData
        }).then(data => {
          if (data.data === null) {
            this.$message.success('保存成功')
            this.isVisible = false
            this.$emit('ok', data.data)
          }
        })
      })
    },
    // 上传图片
    uploadImg(e) {
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
        this.$message.warning('图片类型必须是.gif,jpeg,jpg,png,bmp中的一种')
        return false
      }
      const reader = new FileReader()
      reader.onload = e => {
        let data
        if (typeof e.target.result === 'object') {
          // 如果不是base64,则把Array Buffer转化为blob
          data = window.URL.createObjectURL(new Blob([e.target.result]))
        } else {
          data = e.target.result
        }
        this.option.img = data
        this.$refs.uploadImg.value = ''
      }
      reader.readAsArrayBuffer(e.target.files[0])
    },
    cropMoving(data) {
      this.option.cropData = data
    },
    realTime(data) {
      this.previews = data
    }
  }
}
</script>

<style lang="scss" scoped>
.vue-cropper {
  background: #f2f2f2;
}
.container {
  &::after {
    content: '';
    clear: both;
    display: block;
  }
  .upload_file {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    opacity: 0;
  }
  .crop_area {
    float: left;
    height: 300px;
    width: 300px;
  }
  .preview {
    float: right;
    text-align: center;
    width: 152px;
    .preview__img {
      width: 80px;
      height: 80px;
      overflow: hidden;
      border-radius: 80px;
      margin: 40px auto 10px auto;
      background: #f2f2f2;
    }
  }
}
</style>
