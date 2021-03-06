<template>
  <div>
    <el-table
      v-loading.body="listLoading"
      :data="aboutUsList"
      border
      fit
      highlight-current-row
      style="width: 100%">
      <el-table-column align="center" label="ID" width="60px">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column width="140px" align="center" label="Date">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>

      <el-table-column min-width="200px" align="left" label="Heading">
        <template slot-scope="scope">
          <span>{{ scope.row.heading }}</span>
        </template>
      </el-table-column>

      <el-table-column min-width="200px" align="left" label="SubHeading">
        <template slot-scope="scope">
          <span>{{ scope.row.sub_heading }}</span>
        </template>
      </el-table-column>

      <el-table-column width="200px" align="center" label="Picture">
        <template slot-scope="scope">
          <img :src="scope.row.pic" width="100%">
        </template>
      </el-table-column>

      <el-table-column width="220px" align="center" label="Operation">
        <template slot-scope="scope">
          <el-button
            v-if="typeof(permList) !== 'undefined' && permList.indexOf('sys:user:update') !== -1"
            size="mini"
            type="primary"
            @click="handleUpdate(scope.row)">编辑
          </el-button>
          <el-button
            v-if="typeof(permList) !== 'undefined' && permList.indexOf('sys:user:ban') !== -1 && scope.row.enable === 1"
            size="mini"
            type="warning"
            @click="handleUpdate(scope.row)">禁用
          </el-button>
          <el-button
            v-else-if="typeof(permList) !== 'undefined' && permList.indexOf('sys:user:ban') !== -1 && scope.row.enable === 0"
            size="mini"
            type="success"
            @click="handleUpdate(scope.row)">启用
          </el-button>
          <el-button
            v-if="typeof(permList) !== 'undefined' && permList.indexOf('sys:user:delete') !== -1"
            size="mini"
            type="danger"
            @click="handleDelete(scope.row)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination
        :current-page="listQuery.page"
        :page-sizes="[5,10,20]"
        :page-size="listQuery.size"
        :total="total"
        background
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"/>
    </div>

    <el-dialog :title="textMap[dialogStatus]" :close-on-click-modal="false" :visible.sync="dialogFormVisible" width="80%">
      <el-form :model="about_us" class="small-space about-us-form" label-position="left" label-width="70px">
        <el-form-item class="about-us-form-item" label="类别">
          <el-select v-model="about_us.type" placeholder="请选择">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"/>
          </el-select>
        </el-form-item>

        <el-form-item class="about-us-form-item" label="启用">
          <el-switch
            v-model="about_us.enable"
            active-color="#13ce66"
            inactive-color="#ff4949"/>
        </el-form-item>

        <el-form-item class="about-us-form-item" label="标题">
          <el-input
            v-model="about_us.heading"
            style="width: 90%;"
            class="filter-item"
            placeholder="请输入"/>
        </el-form-item>

        <el-form-item class="about-us-form-item" label="副标题">
          <el-input
            v-model="about_us.sub_heading"
            style="width: 90%;"
            class="filter-item"
            placeholder="请输入"/>
        </el-form-item>

        <el-form-item style="margin-bottom: 40px;" label="图片">
          <el-upload
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            action="https://jsonplaceholder.typicode.com/posts/"
            list-type="picture-card">
            <i class="el-icon-plus"/>
          </el-upload>
          <el-dialog :visible.sync="picVisible">
            <img :src="about_us.pic" width="100%" alt="">
          </el-dialog>
        </el-form-item>

        <div class="editor-container">
          <Tinymce :height="400" v-model="about_us.text" />
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="create">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
/* eslint-disable semi */

import { mapGetters } from 'vuex'
import { fetchList } from '@/api/official-site/about-us/aboutUs'
import Tinymce from '@/components/Tinymce'

export default {
  name: 'AboutUstabPane',
  components: { Tinymce },
  props: {
    type: {
      type: String,
      default: 'Introduction'
    }
  },
  data() {
    return {
      aboutUsList: null,
      about_us: {
        id: null,
        type: '',
        created_time: '',
        heading: '',
        sub_heading: '',
        text: '',
        pic: '',
        enable: ''
      },
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        size: 5,
        type: this.type
      },
      sortOptions: [{ label: '全部', key: '-1' }, { label: '已启用', key: '1' }, { label: '未启用', key: '0' }],
      textMap: {
        update: '编辑',
        create: '创建'
      },
      dialogStatus: '',
      dialogFormVisible: false,
      picVisible: false,
      options: [{
        value: 'INTRODUCTION',
        label: '企业简介'
      }, {
        value: 'CULTURE',
        label: '企业文化'
      }, {
        value: 'HONOR',
        label: '资质荣誉'
      }]
    }
  },
  computed: {
    ...mapGetters([
      'permList'
    ])
  },
  created() {
    this.getList()
  },
  methods: {
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      this.about_us.pic = file.url;
      this.picVisible = true;
    },
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery).then(response => {
        this.aboutUsList = response.data.items;
        this.total = response.data.total;
        this.listLoading = false;
      })
    },
    handleSizeChange(val) {
      this.listQuery.size = val;
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val;
      this.getList()
    },
    handleFilter() {
      this.getList()
    },
    handleCreate() {
      this.dialogStatus = 'create';
      this.dialogFormVisible = true
    },
    handleUpdate() {
      this.dialogStatus = 'update';
      this.dialogFormVisible = true
    },
    handleDelete() {},
    handleDownload() {

    },
    update() {},
    create() {}
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .header-container{
    padding-bottom: 10px;
  }

  .about-us-form {
    width: 92%;
    margin-left: 4%;
    .about-us-form-item {
      display: inline-block;
      width: 100%;
    }
  }
  .editor-container {
    min-height: 500px;
    margin: 0 0 30px;
    .editor-upload-btn-container {
      text-align: right;
      margin-right: 10px;
      .editor-upload-btn {
        display: inline-block;
      }
    }
  }
</style>
