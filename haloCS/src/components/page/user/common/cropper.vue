<template>
  <div class="crop"  v-loading.fullscreen.lock="loading" element-loading-text="上传头像中...">

    <div class="now_pro">
      <h3 class="title">当前头像</h3>
      <img :src="avatar" width="100px" style="width:100px;height: 100px;margin-left: 10px">
    </div>
    <div id="demo">
      <h3 class="title">设置新头像</h3>
      <div>
        <div class="show" style="margin-left: 10px">
          <div class="picture" :style="'backgroundImage:url('+headerImage+')'"></div>
        </div>
        <div class="btn_box">
          <el-button class="choose_btn">选择图片</el-button>
          <input type="file" id="change" class="change" accept="image" @change="change" name="imgFile">
        </div>
        <div class="btn_ok">
          <el-button type="primary" class="choose_btn" style="margin-bottom: 5px" @click="save">保存</el-button>
          <br>
          <el-button class="choose_btn" @click='backToIndex'>取消</el-button>
        </div>

      </div>
      <!-- 遮罩层 -->
      <div class="container" v-show="panel">
        <div>
          <img id="image" :src="url" alt="Picture" width="300px" class="imgs">
        </div>

        <el-button type="primary" id="button" @click="crop" style="width: 100px;">裁剪</el-button>
        <div class="container_box">
          <el-button class="choose_btn">重选</el-button>
          <input type="file" class="change" accept="image" @change="change">
        </div>
      </div>


    </div>
  </div>
</template>

<script>
  import Cropper from 'cropperjs';
  import qs from 'qs';

  export default {
    components: {},
    data() {
      return {
        headerImage: '',
        picValue: '',
        cropper: '',
        croppable: false,
        panel: false,
        url: '',
        file: {},
        avatar: "",
        loading:false
      }
    },
    mounted() {
      //初始化这个裁剪框
      var self = this;
      var image = document.getElementsByClassName("imgs")
      this.cropper = new Cropper(image[0], {
        aspectRatio: 1,
        viewMode: 1,
        background: true,
        zoomable: true,
        ready: function () {
          self.croppable = true;
        }
      });
      this.cropper = new Cropper(image[1], {
        aspectRatio: 1,
        viewMode: 1,
        background: true,
        zoomable: true,
        ready: function () {
          self.croppable = true;
        }
      });
    },
    methods: {
      backToIndex(){
        this.$emit('edit', '1');
      },
      getData() {
        var url = this.$rootUrl + "/api/user/userData";
        var token = sessionStorage.getItem('accessToken');
        const options = {
          method: 'POST',
          url: url,
          data: {
            token:token
          }
        };
        this.$axios(options).then((res) => {
          let item = res.data.data;
            if (item.errorCode == 0) {
              var avatar = item.data.userinfo.avatar;
              if (avatar == "//" || avatar == "") {
                this.avatar = "https://image-res.mzres.com/img/download/uc/11/03/57/90/00/11035790/w100h100?t=1556275801"
              }
              else {
                this.avatar = avatar;
              }
            }else if (item.errorCode == 403) {
              sessionStorage.setItem('pageHistory', this.$route.fullPath);
              this.$router.push({path: "/login"});
              throw item.msg;
            } else {
              throw item.msg;
            }
      }).catch(errorMsg => {
          this.$message.error(errorMsg);
      });
      },
      save() {
        this.loading = true;
        let param = {
          img:this.headerImage
        };
        var url = this.$rootUrl + "/api/user/saveAvatar";

        this.$axios.post(url, param).then((res) => {
          let item = res.data.data;
            if (item.errorCode == 0) {
              if (item.data.url) {
                this.$message({
                  message: '修改头像成功',
                  type: 'success'
                });
//                this.getData();
                this.headerImage = '';
                sessionStorage.setItem('avatar',item.data.url);
                this.backToIndex();
              }
            }else if (item.errorCode == 403) {
              sessionStorage.setItem('pageHistory', this.$route.fullPath);
              this.$router.push({path: "/login"});
              throw item.msg;
            } else {
              throw item.msg;
            }
          this.loading = false;
        }).catch(errorMsg => {
          this.$message.error(errorMsg);
        });
      },
      getObjectURL(file) {
        var url = null;
        if (window.createObjectURL != undefined) { // basic
          url = window.createObjectURL(file);
        } else if (window.URL != undefined) { // mozilla(firefox)
          url = window.URL.createObjectURL(file);
        } else if (window.webkitURL != undefined) { // webkit or chrome
          url = window.webkitURL.createObjectURL(file);
        }
        return url;
      },
      change(e) {
        this.file = e.target.files[0]
        let files = e.target.files || e.dataTransfer.files;
        if (!files.length){
          return;
        }
        if( (files.size/1024) > 5 ){
          this.$message.error('图片尺寸超过5M，请重新选择');
          return;
        }
        this.panel = true;
        this.picValue = files[0];

        this.url = this.getObjectURL(this.picValue);
        //每次替换图片要重新得到新的url
        if (this.cropper) {
          this.cropper.replace(this.url);
        }
        this.panel = true;

      },
      crop() {
        this.panel = false;
        var croppedCanvas;
        var roundedCanvas;


        if (!this.croppable) {
          return;
        }
        // Crop
        croppedCanvas = this.cropper.getCroppedCanvas();
        // Round
        roundedCanvas = this.getRoundedCanvas(croppedCanvas);

        this.headerImage = roundedCanvas.toDataURL();
        console.log(this.headerImage);
        this.postImg()

      },
      getRoundedCanvas(sourceCanvas) {

        var canvas = document.createElement('canvas');
        var context = canvas.getContext('2d');
        var width = sourceCanvas.width;
        var height = sourceCanvas.height;

        canvas.width = width;
        canvas.height = height;

        context.imageSmoothingEnabled = true;
        context.drawImage(sourceCanvas, 0, 0, width, height);
        context.globalCompositeOperation = 'destination-in';
        context.beginPath();
        context.rect(0, 0, Math.min(width, height), Math.min(width, height));
        context.fill();

        return canvas;
      },
      postImg() {
      }
    },
    created() {
      this.getData()
    }
  }
</script>

<style>
  .crop {
    margin-left: 5px;
  }

  .title {
    color: #666666;
    font-size: 18px;
    margin-top: 30px;
    margin-bottom: 10px;
    padding-bottom: 10px;
    padding-left: 10px;
    border-bottom: 0.5px solid #dcdcdc;
  }

  #demo #button {
    position: relative;
    right: 00px;
    top: 10px;
  }

  #demo .show {
    width: 100px;
    height: 100px;
    overflow: hidden;
    position: relative;
    /*border-radius: 50%;*/
    border: 1px solid #d5d5d5;
  }

  #demo .picture {
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-position: center center;
    background-repeat: no-repeat;
    background-size: cover;
  }

  #demo .container {
    width: 300px;
    overflow: hidden;
    z-index: 99;
    position: relative;
    left: 260px;
    top: -280px;
    right: 0;
    bottom: 0;
    background: #fff;
    margin-bottom: 100px;
    padding-bottom: 50px;
  }

  #demo #image {
    width: 100%;
    max-width: 100%;
  }

  .btn_box {
    position: relative;
    margin-top: 10px;
    margin-left: 10px;
    margin-bottom: 20px;
  }

  .choose_btn {
    width: 100px !important;
  }

  .btn_ok {
    margin-bottom: 50px;
    margin-left: 10px;
  }

  .container_box {
    position: relative;
    left: 120px;
    top: -21px;
  }

  .change {
    width: 80px;
    height: 30px;
    opacity: 0;
    position: absolute;
    left: 0px;
    cursor: pointer;
  }

  .cropper-view-box, .cropper-face {
    border-radius: 50%;
  }

  .cropper-container {
    font-size: 0;
    line-height: 0;

    position: relative;

    -webkit-user-select: none;

    -moz-user-select: none;

    -ms-user-select: none;

    user-select: none;

    direction: ltr;
    -ms-touch-action: none;
    touch-action: none
  }

  .cropper-container img {
    /* Avoid margin top issue (Occur only when margin-top <= -height) */
    display: block;
    min-width: 0 !important;
    max-width: none !important;
    min-height: 0 !important;
    max-height: none !important;
    width: 100%;
    height: 100%;
    image-orientation: 0deg
  }

  .cropper-wrap-box,
  .cropper-canvas,
  .cropper-drag-box,
  .cropper-crop-box,
  .cropper-modal {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }

  .cropper-wrap-box {
    overflow: hidden;
  }

  .cropper-drag-box {
    opacity: 0;
    background-color: #fff;
  }

  .cropper-modal {
    opacity: .5;
    background-color: #000;
  }

  .cropper-view-box {
    display: block;
    overflow: hidden;

    width: 100%;
    height: 100%;

    outline: 1px solid #39f;
    outline-color: rgba(51, 153, 255, 0.75);
  }

  .cropper-dashed {
    position: absolute;

    display: block;

    opacity: .5;
    border: 0 dashed #eee
  }

  .cropper-dashed.dashed-h {
    top: 33.33333%;
    left: 0;
    width: 100%;
    height: 33.33333%;
    border-top-width: 1px;
    border-bottom-width: 1px
  }

  .cropper-dashed.dashed-v {
    top: 0;
    left: 33.33333%;
    width: 33.33333%;
    height: 100%;
    border-right-width: 1px;
    border-left-width: 1px
  }

  .cropper-center {
    position: absolute;
    top: 50%;
    left: 50%;

    display: block;

    width: 0;
    height: 0;

    opacity: .75;
  }

  .cropper-center:before,
  .cropper-center:after {
    position: absolute;
    display: block;
    content: ' ';
    background-color: #eee
  }

  .cropper-center:before {
    top: 0;
    left: -3px;
    width: 7px;
    height: 1px
  }

  .cropper-center:after {
    top: -3px;
    left: 0;
    width: 1px;
    height: 7px
  }

  .cropper-face,
  .cropper-line,
  .cropper-point {
    position: absolute;

    display: block;

    width: 100%;
    height: 100%;

    opacity: .1;
  }

  .cropper-face {
    top: 0;
    left: 0;

    background-color: #fff;
  }

  .cropper-line {
    background-color: #39f
  }

  .cropper-line.line-e {
    top: 0;
    right: -3px;
    width: 5px;
    cursor: e-resize
  }

  .cropper-line.line-n {
    top: -3px;
    left: 0;
    height: 5px;
    cursor: n-resize
  }

  .cropper-line.line-w {
    top: 0;
    left: -3px;
    width: 5px;
    cursor: w-resize
  }

  .cropper-line.line-s {
    bottom: -3px;
    left: 0;
    height: 5px;
    cursor: s-resize
  }

  .cropper-point {
    width: 5px;
    height: 5px;

    opacity: .75;
    background-color: #39f
  }

  .cropper-point.point-e {
    top: 50%;
    right: -3px;
    margin-top: -3px;
    cursor: e-resize
  }

  .cropper-point.point-n {
    top: -3px;
    left: 50%;
    margin-left: -3px;
    cursor: n-resize
  }

  .cropper-point.point-w {
    top: 50%;
    left: -3px;
    margin-top: -3px;
    cursor: w-resize
  }

  .cropper-point.point-s {
    bottom: -3px;
    left: 50%;
    margin-left: -3px;
    cursor: s-resize
  }

  .cropper-point.point-ne {
    top: -3px;
    right: -3px;
    cursor: ne-resize
  }

  .cropper-point.point-nw {
    top: -3px;
    left: -3px;
    cursor: nw-resize
  }

  .cropper-point.point-sw {
    bottom: -3px;
    left: -3px;
    cursor: sw-resize
  }

  .cropper-point.point-se {
    right: -3px;
    bottom: -3px;
    width: 20px;
    height: 20px;
    cursor: se-resize;
    opacity: 1
  }

  @media (min-width: 768px) {

    .cropper-point.point-se {
      width: 15px;
      height: 15px
    }
  }

  @media (min-width: 992px) {

    .cropper-point.point-se {
      width: 10px;
      height: 10px
    }
  }

  @media (min-width: 1200px) {

    .cropper-point.point-se {
      width: 5px;
      height: 5px;
      opacity: .75
    }
  }

  .cropper-point.point-se:before {
    position: absolute;
    right: -50%;
    bottom: -50%;
    display: block;
    width: 200%;
    height: 200%;
    content: ' ';
    opacity: 0;
    background-color: #39f
  }

  .cropper-invisible {
    opacity: 0;
  }

  .cropper-bg {
    background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC');
  }

  .cropper-hide {
    position: absolute;

    display: block;

    width: 0;
    height: 0;
  }

  .cropper-hidden {
    display: none !important;
  }

  .cropper-move {
    cursor: move;
  }

  .cropper-crop {
    cursor: crosshair;
  }

  .cropper-disabled .cropper-drag-box,
  .cropper-disabled .cropper-face,
  .cropper-disabled .cropper-line,
  .cropper-disabled .cropper-point {
    cursor: not-allowed;
  }


</style>
