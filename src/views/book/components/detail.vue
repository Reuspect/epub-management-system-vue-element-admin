<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <Sticky :class-name="'sub-navbar '+ postForm.status">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button
        v-loading="loading"
        type="success"
        style="margin-left: 10px"
        @click="submitForm"
      >{{ isEdit? '编辑电子书': '新增电子书' }}</el-button>
    </Sticky>
    <div class="detail-container">
      <el-row>
        <Warning />
        <el-col :span="24">
          <!-- 上传表单控件的具体样式 -->
          <EbookUpload
            :file-list="fileList"
            :disabled="isEdit"
            @onSuccess="onUploadSuccess"
            @onRemove="onUploadRemove"
          />
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title">
            <MdInput
              v-model="postForm.title"
              :maxlength="100"
              name="name"
              required
            >
              书名
            </MdInput>
          </el-form-item>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="author" label="作者：" :label-width="labelWidth">
                <el-input v-model="postForm.author" placeholder="作者" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="publisher" label="出版社：" :label-width="labelWidth">
                <el-input v-model="postForm.publisher" placeholder="出版社" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="language" label="语言：" :label-width="labelWidth">
                <el-input v-model="postForm.language" placeholder="语言" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="rootFile" label="根文件：" :label-width="labelWidth">
                <el-input v-model="postForm.rootFile" placeholder="根文件" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="filePath" label="文件路径：" :label-width="labelWidth">
                <el-input v-model="postForm.filePath" placeholder="文件路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="unzipPath" label="解压路径：" :label-width="labelWidth">
                <el-input v-model="postForm.unzipPath" placeholder="解压路径" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="coverPath" label="封面路径：" :label-width="labelWidth">
                <el-input v-model="postForm.coverPath" placeholder="封面路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="originalName" label="文件名称：" :label-width="labelWidth">
                <el-input v-model="postForm.originalName" placeholder="文件名称" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item prop="cover" :label-width="labelWidth" label="封面：">
                <a
                  v-if="postForm.cover"
                  :href="postForm.cover"
                  target="_blank"
                >
                  <img :src="postForm.cover" class="preview-img">
                </a>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item :label-width="labelWidth" label="目录：">
                <div v-if="contentsTree && contentsTree.length > 0" class="contents-wrapper">
                  <el-tree :data="contentsTree" @node-click="onContentClick" />
                </div>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>

<script>
import Sticky from '../../../components/Sticky/index.vue'
import Warning from './Warning.vue'
import EbookUpload from '../../../components/EbookUpload'
import MdInput from '../../../components/MDinput'
import { createBook, getBook, updateBook } from '../../../api/book'

// const defaultForm = {
//   title: '', author: '', publisher: '', language: '', rootFile: '', cover: '', url: '', originalName: '',
//   fileName: '', coverPath: '', filePath: '', unzipPath: ''
// }

const fields = {
  title: '书名',
  author: '作者',
  publisher: '出版社',
  language: '语言'
}
export default {
  components: {
    Sticky,
    Warning,
    MdInput,
    EbookUpload
  },
  props: {
    isEdit: Boolean
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      // 不管成功失败都要回调一下callback 这里校验规则是必须填写 长度不为0
      console.log(rule, value)
      if (value.length === 0) {
        callback(new Error(fields[rule.field] + '必须填写'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      postForm: {
        // status是控制上面的class显示不同颜色的
      },
      fileList: [],
      labelWidth: '120px',
      contentsTree: [],
      rules: {
        // key是el-form-item对应的prop属性
        title: [{ validator: validateRequire }], // 每一个prop作为key的值都是数组 包含一系列的校验对象,每一项对应一个校验规则
        author: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }]
      }
    }
  },
  created() {
    if (this.isEdit) {
      const fileName = this.$route.params.fileName
      this.getBookData(fileName) // 主要负责调用接口获取书本的信息
    }
  },
  methods: {
    getBookData(fileName) { // 新建getBook的api
      getBook(fileName).then(response => {
        this.setData(response.data)
      })
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text) // 在新窗口打开(返回的就是url)
      }
    },
    setData(data) {
      // 用途 更新我们的表单数据到postForm
      const { title, author, publisher, language, rootFile, cover, url, originalName,
        contents, contentsTree, fileName, coverPath, filePath, unzipPath } = data
      this.postForm = {
        ...this.postForm,
        title, author, publisher, language, rootFile, cover, url, originalName,
        contents, fileName, coverPath, filePath, unzipPath
      }
      this.fileList = [{ name: originalName || fileName, url }] // 主要是在编辑电子书界面 获取到一本电子书时，显示其文件名称
      this.contentsTree = contentsTree
    },
    setDefault() {
      this.contentsTree = []
      this.$refs.postForm.resetFields() // 可以给所有有prop属性的全部清空 且不做校验
      // this.postForm = Object.assign({}, defaultForm) // 赋一个默认值
    },
    onUploadSuccess(data) {
      console.log('onUploadSuccess', data)
      this.setData(data)
    },
    onUploadRemove() {
      console.log('onUploadRemove')
      this.setDefault()
      this.contentsTree = []
    },
    showGuide() {
      this.loading = true
      setTimeout(() => {
        this.loading = false
      }, 1000)
      console.log('showGuide')
    },
    submitForm() {
      const onSuccess = (response) => { // 食用箭头函数防止this无法取到实例
        const { msg } = response
        this.$notify({
          title: '操作成功',
          message: msg,
          type: 'success',
          duration: 2000
        })
        this.loading = false
        this.setDefault()
      }
      if (!this.loading) {
        this.loading = true
        console.log('submitForm')
        this.$refs.postForm.validate((valid, fields) => {
          console.log(valid, fields) // 需要先给el-form字段绑定:rules 如果不符合的话valid才会变成false
          if (valid) {
            // 首先做一个浅拷贝
            const book = Object.assign({}, this.postForm) // 方法1
            // const book = {...this.postForm} // 方法2
            // delete book.contents // 一开始没有处理book.contents的数据库插入
            delete book.contentsTree
            if (!this.isEdit) {
              createBook(book).then(response => {
                onSuccess(response)
              })
                .catch(() => {
                  this.loading = false
                }) // 对应两个页面 都需要调用api
            } else { // 上面是创建电子书 下面是更新电子书
              updateBook(book).then(response => {
                onSuccess(response)
              }).catch(() => {
                this.loading = false
              })
            }
          } else {
            // fields[Object.keys(fields)[0]] //这是个error对象 第一个元素正是title
            const message = fields[Object.keys(fields)[0]][0].message // 就为了拿到”标题必须填写“
            this.$message({ message, type: 'error' })
            this.loading = false
          }
        }) // 源码位于element-ui-package-form 最后callback(valid, invalidFields)来回调显示我们的判断是否正确
      }
    }
  }
}
</script>
<style lang='scss' scoped>
  .detail-container {
    padding: 40px 50px 20px;
    .preview-img {
      width: 200px;
      height: 270px;
    }
  }
</style>
