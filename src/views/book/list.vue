<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.title"
        placeholder="书名"
        style="width: 200px"
        class="filter-item"
        clearable
        @keyup.enter.native="handleFilter"
        @clear="handleFilter"
        @blur="handleFilter"
      />
      <el-input
        v-model="listQuery.author"
        placeholder="作者"
        style="width: 200px"
        class="filter-item"
        clearable
        @keyup.enter.native="handleFilter"
        @clear="handleFilter"
        @blur="handleFilter"
      />
      <el-select
        v-model="listQuery.category"
        placeholder="分类"
        clearable=""
        class="filter-item"
        @change="handleFilter"
      >
        <el-option
          v-for="item in categoryList"
          :key="item.value"
          :label="item.label+'('+item.num+')'"
          :value="item.value"
        />
      </el-select>
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        style="margin-left: 10px"
        @click="handleFilter"
      >查询
      </el-button>
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        style="margin-left: 5px"
        @click="handleCreate"
      >新增
      </el-button>
      <el-checkbox
        v-model="showCover"
        class="filter-item"
        style="margin-left: 5px"
        @change="changeShowCover"
      >
        显示封面
      </el-checkbox>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      fit
      highlight-current-row
      style="width:100%"
      @sort-change="sortChange"
    >
      <el-table-column
        label="ID"
        prop="id"
        sortable="custom"
        align="center"
        width="80"
      />
      <el-table-column
        label="书名"
        align="center"
        width="150"
      >
        <template v-slot="{ row: { titleWrapper }}">
          <span v-html="titleWrapper">{{ titleWrapper }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="作者"
        align="center"
        width="150"
      >
        <template v-slot="{ row: { authorWrapper }}">
          <span v-html="authorWrapper">{{ authorWrapper }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="出版社"
        prop="publisher"
        width="150"
        align="center"
      />
      <el-table-column
        label="分类"
        prop="categoryText"
        width="100"
        align="center"
      />
      <el-table-column
        label="语言"
        prop="language"
        align="center"
      />
      <el-table-column
        v-if="showCover"
        label="封面"
        align="center"
        width="150"
      >
        <template v-slot="scope">
          <a :href="scope.row.cover" target="_blank">
            <img :src="scope.row.cover" style="width:120px; height:180px">
          </a>
        </template>
      </el-table-column>
      <el-table-column
        label="文件名"
        prop="fileName"
        align="center"
        width="100"
      />
      <el-table-column
        label="文件路径"
        prop="filePath"
        align="center"
        width="100"
      >
        <template v-slot="{row: {filePath}}">
          <span>{{ filePath | valueFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="封面路径"
        prop="coverPath"
        align="center"
        width="100"
      >
        <template v-slot="{row: {coverPath}}">
          <span>{{ coverPath | valueFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="解压路径"
        prop="unzipPath"
        align="center"
        width="100"
      >
        <template v-slot="{row: {unzipPath}}">
          <span>{{ unzipPath | valueFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="上传人"
        prop="createUser"
        align="center"
        width="100"
      >
        <template v-slot="{row: {createUser}}">
          <span>{{ createUser | valueFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="上传时间"
        prop="createDt"
        align="center"
        width="100"
      >
        <template v-slot="{row: { createDt }}">
          <span>{{ createDt | timeFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="120"
        fixed="right"
      >
        <template v-slot="{ row }">
          <el-button type='text' icon="el-icon-edit"
           @click="handleUpdate(row)" />
           <el-button type='text' icon="el-icon-delete"
           @click="handleDelete(row)" />
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.pageSize"
      @pagination="getList"
    />
  </div>
</template>
<script>
import Pagination from '../../components/Pagination'
import waves from '../../directive/waves/waves'
import { getCategory, listBook, deleteBook } from '../../api/book'
import { parseTime } from '../../utils'

export default {
  components: { Pagination },
  directives: { waves },
  filters: {
    valueFilter(value) {
      return value || '无'
    },
    timeFilter(time) {
      // 框架自带处理时间的方法
      return time ? parseTime(time, '{y}-{m}-{d} {h}:{i}') : '无'
    }
  },
  data() {
    return {
      tableKey: 0, // 如果存在多个table对数据进行区分
      listLoading: false,
      listQuery: { // 与查询列表绑定
        page: 1,
        pageSize: 20,
        sort: '+id'
      },
      showCover: false, // 用来控制封面是否显示
      categoryList: [], // 这个是用来动态存储option对应的分类选项的
      list: [],
      total: 0
    }
  },
  created() {
    this.parseQuery() // 说明created时候就可以执行方法了
  },
  mounted() {
    this.getList()
    this.getCategoryList()
  },
  methods: {
    parseQuery() {
      const listQuery = {
        page: 1,
        pageSize: 20
      }
      this.listQuery = { ...listQuery, ...this.listQuery } // 相当于用listQuery做一个默认值
    },
    sortChange(data) {
      console.log('sortChange', data)
      const { prop, order } = data
      this.sortBy(prop, order)
    },
    sortBy(prop, order) {
      if (order === "ascending") {
        this.listQuery.sort = `+${prop}`
      } else {
        this.listQuery.sort = `-${prop}`
      }
      this.handleFilter() // 排序完记得做一次请求
    },
    wrapperKeyword(k, v) {
      function highlight(value) {
        return `<span style="color: #1890ff">${value}</span>`
      }
      if (!this.listQuery[k]) { // k可以是listQuery的任何键 如title和author
        return v
      } else {
        return v.replace(new RegExp(this.listQuery[k], 'ig'), v => highlight(v))
      }
    },
    getList() { // 把listQuery作为查询参数传入进去，调用新的接口
      this.listLoading = true
      listBook(this.listQuery).then(response => {
        const { list, count, page, pageSize } = response.data
        this.list = list
        this.total = count
        this.listLoading = false
        this.list.forEach(book => {
          book.titleWrapper = this.wrapperKeyword('title', book.title)
          book.authorWrapper = this.wrapperKeyword('author', book.author)
        })
      })
    },
    getCategoryList() { // 从后端获取动态分类数据 getCategory方法需要请求后端接口 写在api中
      getCategory().then(response => {
        this.categoryList = response.data
      })
    },
    handleFilter() { // 注意点击enter还是清除,失焦都会重新处理查询条件
      console.log('handleFilter', this.listQuery)
      this.getList() // 每次都重新获取一次列表
    },
    handleCreate() {
      this.$router.push('/book/create')
    },
    handleUpdate(row) {
      this.$router.push(`/book/edit/${row.fileName}`)
    },
    handleDelete(row) {
      this.$confirm('此操作将永久删除该电子书，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteBook(row.fileName).then(response => {
          this.$notify({
            title: '成功',
            message: response.msg || '删除成功',
            type: 'success',
            duration: 2000
          })
          this.handleFilter() // 记得刷新列表
        }) // 是api方法
      }).catch(() => {

      }) // elementui提供 生成的confirmbox
      console.log(row)
    },
    changeShowCover(value) { // 会接受一个value
      // console.log(value)
      this.showCover = value // true / false
    }
  }
}
</script>
<style lang="scss" scoped>

</style>
