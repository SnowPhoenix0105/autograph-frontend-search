<template>
  <div class="common-layout">
    <el-container>
      <el-header>
        <div class="head-title">
          知识图谱查询
        </div>
      </el-header>
      <el-container>



<!--        左侧栏       -->
        <el-aside width="200px">
          <el-row>
            <el-col>
              <el-button class="menu-button" size="large" type="success" plain>
                历史记录
              </el-button>
            </el-col>
          </el-row>
            <div v-for="h in history" :key="h">
              <el-row>
                <el-col>
                  <el-button class="menu-button" size="large" type="primary" plain @click="tryVisit(h)">
                    {{ h }}
                  </el-button>
                </el-col>
              </el-row>
            </div>
        </el-aside>

<!--中心-->
        <el-main :style="{
          boxShadow: `dark`,
        }">
          <el-row>
            <el-col
                :offset="2"
                :span="8">
              <el-input
                  v-model="header.input_line"
                  size="large"
                  placeholder="请输入实体名"  />
            </el-col>
            <el-col :span="2">
              <el-button
                  type="primary"
                  size="large"
                  @click="searchHandler" >搜索</el-button>
            </el-col>
            <el-col :span="4"></el-col>
          </el-row>
          <KnowledgeGraph :chartList="kg_data" @onNodeClick="tryVisit"></KnowledgeGraph>

        </el-main>




<!--        右侧栏       -->
        <el-aside width="400px">

          <el-row>
            <el-col>
              <el-button class="file-download" size="large" type="success" plain>
                课件下载
              </el-button>
            </el-col>
          </el-row>
          <div v-for="f in files" :key="f.name">
            <el-row>
              <el-col>
                <el-button class="file-button" size="large" type="primary" plain @click="download(f.id)">
                  <div v-if="f.type === 'doc'">
                    <img class='fileicon' src="/fileicon/word.ico" >
                  </div>
                  <div v-if="f.type === 'txt'">
                    <img class='fileicon' src="/fileicon/txt.ico" >
                  </div>
                  <div v-if="f.type === 'pptx'">
                    <img class='fileicon' src="/fileicon/ppt.ico" >
                  </div>
                  {{ f.name }}
                </el-button>
              </el-col>
            </el-row>
          </div>

        </el-aside>
      </el-container>
    </el-container>
  </div>
</template>

<script>


import KnowledgeGraph from "@/components/KnowledgeGraph";
import {ElMessage} from "element-plus";
export default {
  name: "MainContainer",
  components: {KnowledgeGraph},
  data() {
    return {
      header: {
        input_line: "",
      },
      kg_data: {},
      history: [],
      files: []
    }
  },
  methods: {
    async tryVisit(nodeName) {
      let res = await this.$axios.get(this.$searchServ + "/search?q=" + encodeURI(nodeName))

      if (res.status !== 200) {
        console.warn(res)
        return
      }

      this.history.unshift(nodeName)

      // console.dir(res.data)

      if (res.data.code !== 0) {
        console.warn(res.data.message)
        return
      }

      if (!res.data.data.graph.nodes) {
        ElMessage.warning("无搜索结果")
        return
      }

      this.kg_data = res.data.data

      let new_files = []
      res.data.data.files.forEach((f) => {
        new_files.push({
          name: f.file_name,
          type: f.file_type,
          id: f.file_id,
        })
      })

      this.files = new_files
    },

    async searchHandler() {
      const query = this.header.input_line
      // console.log("query=", query)
      await this.tryVisit(query)
    },

    async download(fileID) {
      const resp = await this.$axios.get(
          this.$searchServ + "/download?id=" + fileID,
          {
            responseType: "blob",
          }
      );

      console.log("response");
      console.dir(resp);

      const fileName =
          resp.headers["content-disposition"].match(/filename=(.*)/)[1];

      // 将二进制流转为blob
      const blob = new Blob([resp.data], {
        type: "application/octet-stream",
      });

      if (typeof window.navigator.msSaveBlob !== "undefined") {
        // 兼容IE，window.navigator.msSaveBlob：以本地方式保存文件
        window.navigator.msSaveBlob(blob, decodeURI(fileName));
        return
      }

      // 创建新的URL并指向File对象或者Blob对象的地址
      const blobURL = window.URL.createObjectURL(blob);
      // 创建a标签，用于跳转至下载链接
      const tempLink = document.createElement("a");
      tempLink.style.display = "none";
      tempLink.href = blobURL;
      tempLink.setAttribute("download", decodeURI(fileName));
      // 兼容：某些浏览器不支持HTML5的download属性
      if (typeof tempLink.download === "undefined") {
        tempLink.setAttribute("target", "_blank");
      }
      // 挂载a标签
      document.body.appendChild(tempLink);
      tempLink.click();
      document.body.removeChild(tempLink);
      // 释放blob URL地址
      window.URL.revokeObjectURL(blobURL);

    },
  }
}
</script>



<style scoped>
.head-title {
  font-size: 58px;
  font-family: 黑体;
  color: azure;
}
.el-header {
  position: relative;
  background-color: #0088ff;
  color: var(--el-text-color-primary);
  height: 70px;
}
.el-aside {
  color: var(--el-text-color-primary);
  background-color: #f0ffff;
  height: calc(100vh - 100px);
  text-align: center;
}
.menu-button {
  padding: 30px;
  width: 160px;
  height: 50px;
  margin-top: 10px;
  margin-left: 20px;
  margin-right: 20px;
}
.file-download {
  padding: 30px;
  width: 360px;
  height: 50px;
  margin-top: 10px;
  margin-left: 20px;
  margin-right: 20px;
}
.file-button {
  padding: 30px;
  width: 360px;
  height: 50px;
  margin-top: 10px;
  margin-left: 20px;
  margin-right: 20px;
  text-align: left;
  justify-content: left;
}
.fileicon {
  height: 40px;
  width: 40px;
  margin-right: 20px;
}
</style>