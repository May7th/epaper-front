<template>
  <div style="margin: 20px;">
    <el-row :gutter="10" type="flex">
      <el-col :span="1.5">
        <p class="label">文章标题</p>
      </el-col>
      <el-col :span="1.5">
        <mongolian-input v-model="article.title" ref="title" style="height: 590px;"></mongolian-input>
      </el-col>
      <el-col :span="1.5">
        <p class="label">副标题</p>
      </el-col>
      <el-col :span="1.5">
        <mongolian-input v-model="article.subTitle" ref="subTitle" style="height: 590px;"></mongolian-input>
      </el-col>
      <el-col :span="1.5">
        <p class="label">作者</p>
      </el-col>
      <el-col :span="1.5">
        <mongolian-input v-model="article.author" ref="author" style="height: 590px;"></mongolian-input>
      </el-col>
      <el-col :span="1.5">
        <p class="label">正文</p>
        <el-button @click="checkContent" type="primary" size="mini" plain style="padding: 8px"><svg-icon icon-class="check" /><p style="writing-mode: vertical-lr">校正</p></el-button>
      </el-col>
      <el-col :span="20">
        <vue-editor v-model="article.contentHtml" useCustomImageHandler
                    @imageAdded="handleImageAdded" style="height: 85%;"
        ></vue-editor>
      </el-col>
      <el-col :span="15">
        <leaflet :pageImagePath="this.pageImagePath" @draw="getCoordinate"></leaflet>
      </el-col>
    </el-row>
    <div>
      文章分类:
      <el-select v-model="article.catalog.id" filterable placeholder="请选择">
        <el-option
          v-for="item in catalogOptions"
          :key="item.id"
          :label="item.name"
          :value="item.id">
        </el-option>
      </el-select>
    </div>

    <div style="margin: 5px;">
      <el-button @click="dialogFormVisible = false">取 消</el-button>
      <el-button type="primary" @click="handleAdd">确 定</el-button>
    </div>

  </div>
</template>

<script>
  import store from '../../store'
  import { addArticle,getCatalogList,checkContent} from '@/api/article'
  import { getPageImage } from '@/api/page'
  import { VueEditor, Quill } from "vue2-editor";
  import Leaflet from '../../components/article/index'
  import MongolianInput from '../../components/MongolianInput/index'
  import axios from 'axios'

  export default {
    name: "",
    components: {
      VueEditor,
      Leaflet,
      MongolianInput
    },
    computed: {
      headers() {
        return {
          'Authorization': 'bearer ' + store.state.user.token
        }
      }
    },
    props: {
      pageId: {
        required: true
      }
    },
    data() {
      return {
        pageImagePath: this.$store.state.article.pageImagePath,
        article: {
          title: '',
          subTitle: '',
          author: '',
          coordinate: '',
          content: '',
          contentHtml: '',
          parentId: this.pageId,
          catalog: {
            id: ''
          }
        },
        catalogOptions: [],
      }
    },
    methods: {
      handleAdd() {
        this.$refs.title.handleInput()
        this.$refs.author.handleInput()
        this.$refs.subTitle.handleInput()
        if (this.article.catalog.id == ""){
          delete this.article.catalog
        }
        return new Promise((resolve, reject) => {
          addArticle(this.article).then(response => {
            this.$message({
              message: response.data.message,
              type: 'success'
            });
            this.$router.go(-1)
            resolve()
          }).catch(error => {
            reject(error)
            this.$message.error('不好意思，出错了哦~');
          })
        })
      },
      getCoordinate(drawItem) {
        this.article.coordinate = JSON.stringify(drawItem);
      },
      handleImageAdded: function(file, Editor, cursorLocation, resetUploader) {
        // An example of using FormData
        // NOTE: Your key could be different such as:
        // formData.append('file', file)

        var formData = new FormData();
        formData.append('file', file)

        axios({
          url: '/upload/',
          method: 'POST',
          data: formData,
          headers: {
            'Authorization': 'bearer ' + store.state.user.token
          }
        })
          .then((result) => {
            console.log(result)
            let url = result.data.data // Get url from response
            Editor.insertEmbed(cursorLocation, 'image', url);
            resetUploader();
          })
          .catch((err) => {
            console.log(err);
          })
      },
      checkContent(){
        checkContent(this.article.contentHtml).then(response => {
          this.article.contentHtml = response.data.data
          this.$message({
            message: "校正成功，内容已替换。",
            type: 'success'
          });
        }).catch(error => {
          console.log(error)
          this.$message.error("校正服务出错，请稍后再试。");
        })
      }
    },
    mounted(){
        getPageImage(this.pageId).then(response=>{
          this.$store.commit('setPageImagePath',response.data)
        }),
        getCatalogList().then(response=>{
          var catalogs = response.data.data
          catalogs.forEach(catalog => {
            this.catalogOptions.push(catalog)
          })
          console.log(response)
        })
    }
  }
</script>

<style>
  .ql-editor {
    writing-mode: vertical-lr;
    font-family: OyunQaganTig;
    max-height: 450px;
  }
  .ql-container{
    overflow-x: auto;
  }
  .label{
    writing-mode: vertical-lr;
    padding-top: 10px;
    color: #000;
    font-size: 15px;
  }
</style>
