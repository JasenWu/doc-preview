<template>
  <section>
    <el-dialog
      :append-to-body="true"
      :close-on-click-modal="false"
      :close-on-press-escape="true"
      :visible.sync="dialogVisible"
      :before-close="handleClose"
      title="附件预览"
      top="25px"
      width="95%"
      custom-class="layout-preview-attachment"
    >
      <span>
        <div
          :style="{height:h +'px'}"
          class="layout-img-preview">
          <a
            v-show="index > 0"
            class="i-pre"
            href="javascript:;"
            title="pre"
            @click="goPre()"><i class="alifont af-zuojiantou" /></a>
          <a
            v-show="attachments && index < (attachments.length -1)"
            class="i-next"
            href="javascript:;"
            title="next"
            @click="goNext()"><i class="alifont af-iconfontjiantou4" /></a>
          <div
            v-if="showImageApp(fileType)"
            class="i-operater">
            <i
              id="download"
              alt="下载"
              title="下载"
              class="i-operater-icon alifont af-clouddownload"
              @click="downLoad()" />
            <i
              alt="旋转"
              title="旋转"
              class="i-operater-icon alifont af-rotateleft_b"
              @click="rotate()" />
            <p class="i-title">{{ curItem.name }}</p>
          </div>
          <div
            :style="{'height':h-60+'px',width:'100%'}"
            class="i-img-view-wrap" >
            <!-- 图像预览 -->
            <img
              v-if="showImageApp(fileType)"
              :style="{width:imgW,height:imgH,transform:'rotateZ('+ rotateVal +'deg)'}"
              :src="curItem.url"
              class="i-img-view" >
            <!-- office 文档 -->
            <iframe
              v-if="showOfficeApp(fileType)"
              :src="'https://view.officeapps.live.com/op/view.aspx?src='+curItem.url"
              :height="h +'px'"
              width="100%"
              frameborder="1"
            />
            <!-- pdf 预览 -->
            <div
              v-if="fileType === 'pdf'"
              id="example1"
              :style="{height:h +'px'}"
            />
          </div>
        </div>
      </span>
    </el-dialog>
  </section>
</template>
<script>

import PDFObject from 'pdfobject'

export default {
  data () {
    return {
      curItem: {},
      index: 0,
      dialogVisible: false,
      h: 0,
      fileType: 'pdf',
      rotateVal: 0,
      imgH: 'auto',
      imgW: 'auto',
      attachments: []
    }
  },
  mounted () {
    this.h = window.innerHeight - (46 + 25 + 20 + 25)
  },
  methods: {
    openPDF (curItem) {
      let type = this.fileType = this.getType(this.curItem)
      if (type !== 'pdf') {
        return false
      }
      var options = {
        height: this.h + 'px',
        pdfOpenParams: { view: 'FitV', page: '0' },
        name: 'mans',
        fallbackLink: '<p>您的浏览器暂不支持此pdf，请下载最新的浏览器</p>'
      }
      this.$timeout(600).then(() => {
        PDFObject.embed(
          curItem.url,
          '#example1',
          options
        )
      })
    },
    goPre () {
      let index = this.index
      if (index > 0) {
        this.index = index -= 1
        this.curItem = this.attachments[index]

        this.openPDF(this.curItem)
      }
      console.log('cur', this.curItem)
    },
    goNext () {
      let len = this.attachments.length
      let index = this.index
      if (index < (len - 1)) {
        this.index = index += 1
        this.curItem = this.attachments[index]
        this.openPDF(this.curItem)
      }
      console.log('cur', this.curItem)
    },
    rotate () {
      let val = this.rotateVal -= 90
      let landscape = (val % 180 === 0)
      let ifLandImg = new Promise((resolve, reject) => {
        let img = new Image()
        img.src = this.curItem.url
        img.onload = () => {
          let w = img.width
          let h = img.height
          let land = w > h// 是否为横向大
          resolve(land)
        }
      })
      ifLandImg.then((ifland) => {
        if (ifland) { // 横向图片、方图
          if (landscape) {
            this.imgW = 'auto'
            this.imgH = 'auto'
          } else {
            this.imgW = this.h - 65 + 'px'
            this.imgH = 'auto'
          }
        } else { // 纵向图片
          if (landscape) {
            this.imgW = this.h - 65 + 'px'
            this.imgH = 'auto'
          } else {
            this.imgW = 'auto'
            this.imgH = 'auto'
          }
        }
      })
    },
    downLoad () {
      window.open(this.curItem.url, '_blank')// 下载
    },
    showOfficeApp (t) {
      let types = ['doc', 'docx', 'xls', 'xlsx']
      let r = false
      for (let t1 in types) {
        if (t === types[t1]) {
          r = true
          break
        }
      }
      return r
    },
    showImageApp (t) {
      let types = ['png', 'jpg', 'jpeg', 'gif']
      let r = false
      for (let t1 in types) {
        if (t === types[t1]) {
          r = true
          break
        }
      }
      return r
    },
    getType (v) {
      let types = ['doc', 'docx', 'xls', 'xlsx', 'pdf', 'png', 'jpg', 'jpeg', 'gif']
      let res = ''
      for (let t in types) {
        if (v.url.indexOf('.' + types[t]) !== -1) {
          res = types[t]
          break
        }
      }
      return res
    },

    handleOpen (v, attachments, k) {
      console.log('attachments',JSON.stringify(attachments));
      
      attachments.forEach(element => {
        element.url = element.url.replace('//upload', '/upload')
      })

      v.url = v.url.replace('//upload', '/upload')
      this.attachments = attachments
      this.index = k
      this.fileType = this.getType(v)
      // 没有匹配的文件预览器
      if (this.fileType === '') {
        this.$message.error('暂不支持此格式的预览,请联系系统管理员吴兴省！')
        return false
      }
      this.dialogVisible = true
      // pdf预览
      if (this.fileType === 'pdf') {
        this.curItem = v
        this.openPDF(this.curItem)
        return false
      }

      if (this.showOfficeApp(this.fileType)) {
        this.curItem = v
        return
      }
      if (this.showImageApp(this.fileType)) {
        this.curItem = v
      }
    },
    handleClose (done) {
      done()
    }

  }
}
</script>
<style lang="less">
.layout-preview-attachment {
  .el-dialog__header {
    padding: 10px;
    background: none;
  }
  .el-dialog__body {
    padding: 10px;

  }
  .layout-img-preview{
    background: #000;
    padding: 0 40px;
    padding-top: 0;
    text-align: center;
    .i-next,.i-pre{
         text-decoration: none;
         i{
            color:#fff;
            font-size: 30px;
            border:0;
            text-decoration: none;
         }
       }

    .i-pre{
        z-index: 9;
        position: absolute;
        left:15px;
        top:50%;
        margin-top:-20px;
      }
       .i-next{
        z-index: 9;
        position: absolute;
        right:15px;
        top:50%;
        margin-top:-20px;
      }
    .i-operater{
      height: 50px;
      .i-operater-icon{
        display:block;color:white;float:right;margin-right:10px;margin-top:10px;font-size:18px;cursor:pointer;
      }
    }
     .i-title{
        line-height: 46px;
        color:white;
        text-align: center;
      }
    .i-img-view-wrap{
      position: relative;
      .i-img-view{
        display: block;
        max-height: 100%;
        max-width: 100%;
        position: absolute;    top: 0;    right: 0;    bottom: 0;    left: 0;    margin: auto;
      }

    }

  }
}
</style>
