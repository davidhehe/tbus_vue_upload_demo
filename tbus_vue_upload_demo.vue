<template>
  <div>
    <el-button @click="getHtml">获取数据</el-button>
    <div style="width:800px;margin:50px;">
      <div id="editor"></div>
    </div>
  </div>
</template>
<script>
//tbus 编辑器
import { createEditor } from "@tanbo/tbus";
import { Observable } from "rxjs";
import "@tanbo/tbus/bundles/tbus.min.css";

import { getUrl } from "../../utils/api";
import getUpload from "../../utils/upload.js";
import { getToken, getToken1 } from "../../utils/auth";
import axios from "axios";
export default {
  data() {
    return {
      upload: {
        uploadURL: getUrl.UPLOAD_URL,
        headers: getUpload._headers,
        params: {
          loginUserId:
            (getToken1("user_Info") &&
              JSON.parse(getToken1("user_Info")).accountId) ||
            "",
          fileType: 14,
          fileUploadType: 2  // 0图标 1logo 2普通图片 3文件(需要加密的) 4普通文件 5普通视频
        }
      },
      editor: ""
    };
  },
  mounted() {
    const _this = this;
    const h = '<h1><span style="color:#db1b1b">欢迎你使用</span>&nbsp;TBus 富文本编辑器...tttttttttttt4<br></h1><p><br></p>';
    const editor = createEditor("#editor", {
      theme: "dark", //可选 'dark' | 'mac-os' | 'mac-os-dark'，不传即为默认样式
      uploader(type) {
        switch (type) {
          case "video":
            console.log("上传视频");
            return _this.uploadMethod('video/mp4',5);
            break;
          case "image":
            console.log("上传图片");
            return _this.uploadMethod('image/*',2);
            break;
          case "audio":
            return _this.uploadMethod('video/mp3',4);
            console.log("上传音频");
            break;
        }
        return Promise.resolve().then(() => {
          return "";
        });
      },
      contents: h
    });
    this.editor = editor;
    // console.log(editor);
    editor.onChange.subscribe(() => {
      console.log(editor.getContents());
    });
  },
  methods: {
    getHtml() {
      console.log(this.editor.getContents());
    },
    //上传方法（图片、音频、视频）
    uploadMethod(acceptType,fileUploadType) {
      const _this = this;
      const fileInput = document.createElement("input");
      fileInput.setAttribute("type", "file");
      fileInput.setAttribute("accept", acceptType);
      fileInput.style.cssText =
        "position: absolute; left: -9999px; top: -9999px; opacity: 0";
      const promise = new Promise((resolve, reject) => {
        fileInput.addEventListener("change", event => {
          const form = new FormData();
          for (const file of event.target.files) {
            form.append("files", file);
          }
          form.append("loginUserId", _this.upload.params.loginUserId);
          form.append("fileType", 14);
          form.append("fileUploadType", fileUploadType);
          document.body.removeChild(fileInput);
          axios
            .post(_this.upload.uploadURL, form, {
              headers: _this.upload.headers
            })
            .then(res => {
              console.log(res);
              if (res.data.code == 200) {
                resolve(res.data.result.results[0].filePaths);
              }else{
                _this.$message({
                type: 'error',
                message: res.data.message||"上传失败"
              });
              }
            });
        });
      });
      document.body.appendChild(fileInput);
      fileInput.click();
      return promise;
    }
  }
};
</script>
