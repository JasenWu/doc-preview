<template>
  <div class="home">
 
    <a
      v-for="(v1,k1) of attachments"
      :key="k1"
      class="previewAt"
      target="_blank"
      @click="preview(v1,attachments,k1)"
    >{{ v1.name || '查看' }}</a>
    <c-preview  ref="preview" />
  </div>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      attachments: [
        {
          name: "12",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/未命名_20190104080109134.doc",
          status: "done",
          id: 16063,
          size: 28672
        },
        {
          name: "5",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/5_20190104080109153.jpg",
          status: "done",
          id: 16064,
          size: 56944
        },
        {
          name: "1",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/1_20190104080109279.jpg",
          status: "done",
          id: 16065,
          size: 72375
        },
        {
          name: "excel",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/excel_2019010408010951.xlsx",
          status: "done",
          id: 16066,
          size: 47383
        },
        {
          name: "未命名",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/未命名_20190104080109558.docx",
          status: "done",
          id: 16067,
          size: 9887
        },
        {
          name: "pdf",
          url:
            "http://test.oa.innotechx.com/api//upload/201901/pdf_20190104080109633.pdf",
          status: "done",
          id: 16068,
          size: 25051
        }
      ]
    };
  },
  components: {
    "c-preview": require("./previewAttachment.vue").default
  },
  methods:{
    preview(v, attachments, k){
      this.$refs['preview'].handleOpen(v, attachments, k)
    }
  }

};
</script>
<style lang="less" scoped>
.home{
  a{
    display: block;
    padding: 10px;
    border-bottom:1px solid #ccc;
    cursor: pointer;
  }
}
</style>

