<template>
  <div class="mod-purchase">
    <!-- 操作区 -->
    <el-form :inline="true"
             :model="filters">
      <el-form-item>
        <el-input v-model="filters.goodsName"
                  clearable
                  @keydown.enter.native="handleKey"
                  placeholder="请输入关键字"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="filters.goodsNo"
                  clearable
                  @keydown.enter.native="handleKey"
                  placeholder="请输入商品编号"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="filters.goodsType"
                  clearable
                  @keydown.enter.native="handleKey"
                  placeholder="请输入商品分类"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="mini"
                   @click="getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary"
                   size="mini"
                   @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-dropdown>
          <el-button type="danger"
                     size="mini"
                     :disabled="dataListSelections.length <= 0">批量操作</el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>
              <el-button type="text"
                         @click="deleteHandle"
                         size="mini">批量删除</el-button>
            </el-dropdown-item>
            <el-dropdown-item>
              <el-button type="text"
                         @click="multiUseHandle"
                         size="mini">批量上架</el-button>
            </el-dropdown-item>
            <el-dropdown-item>
              <el-button type="text"
                         @click="multiStopUseHandle"
                         size="mini">批量下架</el-button>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </el-form-item>
    </el-form>
    <!-- 表格 -->
    <el-table :data="dataList"
              v-loading="loading"
              @selection-change="selectionChangeHandle"
              max-height="450"
              style="width: 100%;">
      <el-table-column type="selection"
                       header-align="center"
                       align="center"
                       width="50">
      </el-table-column>
      <template v-for="item in tableItems">
        <el-table-column :key="item.prop"
                         :prop="item.prop"
                         :label="item.label"
                         sortable>
        </el-table-column>
      </template>
      <el-table-column label="操作"
                       width="210">
        <template slot-scope="scope">
          <el-button type="primary"
                     size="mini"
                     @click="addOrUpdateHandle(scope.row.id)">编辑</el-button>
          <el-button @click="handleDel(scope.row)"
                     type="danger"
                     size="mini">删除</el-button>
          <el-dropdown style="margin-left:10px;">
            <el-button type="warning"
                       size="mini">审核</el-button>
            <el-dropdown-menu slot="dropdown">
              <template v-if="scope.row.goodsState==='已上架'">
                <el-dropdown-item>
                  <el-button type="text"
                             @click="handleStopUse(scope.row)">下架</el-button>
                </el-dropdown-item>
              </template>
              <el-dropdown-item v-else>
                <el-button type="text"
                           @click="handleUse(scope.row)">上架</el-button>
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle"
                   @current-change="currentChangeHandle"
                   :current-page="pageIndex"
                   :page-sizes="[10, 20, 50, 100]"
                   :page-size="pageSize"
                   :total="totalPage"
                   layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible"
                   ref="addOrUpdate"
                   @success="getDataList">
    </add-or-update>
  </div>
</template>

<script>
import { list, del } from '@/api/purchase.js'
import tableItems from '@/assets/data/purchase.json'
import AddOrUpdate from './add-or-update'
export default {
  name: 'purchase', // 采购进货
  data () {
    return {
      tableItems: tableItems.tableColumn,
      filters: {
        goodsName: '',
        goodsNo: '',
        goodsType: ''
      },
      code: null,
      name: null,
      type: null,
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      loading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  mounted () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.loading = true
      list({
        code: this.filters.goodsNo,
        name: this.filters.goodsName,
        type: this.filters.goodsType,
        pageIndex: this.pageIndex,
        pageSize: this.pageSize
      }).then(res => {
        let data = res.data
        this.dataList = data.data
        this.loading = false
        // this.pageIndex = data.pageIndex
        // this.pageSize = data.pageSize
        // this.totalPage = data.totalPage
      })
    },
    handleKey () {
      this.getDataList()
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 批量删除
    deleteHandle () {
      if (this.dataListSelections.length === 0) {
        this.$message.warning('请选择')
      } else {
        // let multi = this.dataListSelections
        this.$confirm('你确定？', '提示', {}).then(() => {
          // let goodsNo = multi.map(el => el.goodsNo)
          //   axios.post('api/goods/multiDel', {
          //     goodsNo: goodsNo
          //   })
          //     .then(res => {
          //       this.$message.success('删除成功')
          //       multi.map(el => {
          //         let i = this.goods.indexOf(el)
          //         this.goods.splice(i, 1)
          //       })
          //       this.loading = true
          //       this.queryAll()
          //       this.loading = false
          //     })
          //     .catch(err => console.log(err))
          // })

        }).catch(() => {
          this.$message.info('删除取消')
        })
      }
    },
    _multiStateChange (state, cb) {
      if (this.multipleSelection.length === 0) {
        this.$message.warning('请选择')
      } else {
        // let multi = this.multipleSelection
        // let goodsNo = multi.map(el => el.goodsNo)
        // axios.post('api/goods/multiStateChange', {
        //   goodsState: state,
        //   goodsNo: goodsNo
        // })
        //   .then(cb)
        //   .catch(err => console.log(err))
      }
    },
    // 批量上架
    multiUseHandle () {
      this._multiStateChange('已上架', res => {
        this.$message.success('已上架')
        // this.queryAll()
      })
    },
    // 批量下架
    multiStopUseHandle () {

    },
    // 删除
    handleDel (row) {
      console.log(row)
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        center: true
      }).then(() => {
        this.loading = true
        del({
          id: row.id
        }).then(res => {
          let data = res.data
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.loading = false
          this.getDataList()
          console.log(data)
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style scoped>
</style>
