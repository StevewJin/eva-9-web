<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form>
        <el-row :gutter="10">
          <el-col :span="8">
            <el-form-item class="">
              <el-input placeholder="输入组名搜索" v-model="listQuery.name" @keyup.enter.native="getList">
                <el-button slot="append" icon="el-icon-search" @click="getList"></el-button>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="1">
            <el-form-item>
              <el-button type="primary" icon="plus" @click="showCreate" v-if="hasPerm('group:add')">添加
              </el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </div>
    <el-table :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit
              highlight-current-row>
      <el-table-column align="center" label="序号" width="80">
        <template slot-scope="scope">
          <span v-text="getIndex(scope.$index)"> </span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="name" label="组名" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="nickname" label="组所属人" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="创建时间" width="170">
        <template slot-scope="scope">
          <span>{{scope.row.createTime}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="createBy" label="创建人" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="管理" width="200" v-if="hasPerm('article:update')">
        <template slot-scope="scope">
          <el-button type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button type="danger" icon="connect" @click="delGroup(scope.$index)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="listQuery.pageNum"
      :page-size="listQuery.pageRow"
      :total="totalCount"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form class="small-space" :model="tempGroup" label-position="left" label-width="120px"
               style='width: 400px; margin-left:100px;'>
        <el-form-item label="组名称">
          <el-input type="text" v-model="tempGroup.name">
          </el-input>
        </el-form-item>
        <el-form-item label="所属人">
          <el-select v-model="tempGroup.uid" placeholder="请选择">
            <el-option
              v-for="item in userList"
              :key="item.userId"
              :label="item.nickname"
              :value="item.userId">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createGroup">创 建</el-button>
        <el-button type="primary" v-else @click="updateGroup">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        userList: [],
        totalCount: 0, //分页组件--数据总条数
        list: [],//表格的数据
        listLoading: false,//数据加载等待动画
        listQuery: {
          pageNum: 1,//页码
          pageRow: 50,//每页条数
          name: ''
        },
        dialogStatus: 'create',
        dialogFormVisible: false,
        textMap: {
          update: '编辑',
          create: '创建服务器'
        },
        tempGroup: {
          id: "",
          name: "",
          uid: ""
        }
      }
    },
    created() {
      this.getList();
      this.getUserList();
    },
    methods: {
      getList() {
        //查询列表
        if (!this.hasPerm('group:list')) {
          return
        }
        this.listLoading = true;
        this.api({
          url: "/group/listGroup",
          method: "get",
          params: this.listQuery
        }).then(data => {
          this.listLoading = false;
          this.list = data.list;
          this.totalCount = data.totalCount;
        })
      },
      handleSizeChange(val) {
        //改变每页数量
        this.listQuery.pageRow = val
        this.handleFilter();
      },
      handleCurrentChange(val) {
        //改变页码
        this.listQuery.pageNum = val
        this.getList();
      },
      getIndex($index) {
        //表格序号
        return (this.listQuery.pageNum - 1) * this.listQuery.pageRow + $index + 1
      },
      showCreate() {
        //显示新增对话框
        this.tempGroup.uid = "";
        this.tempGroup.name = "";
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
      showUpdate($index) {
        //显示修改对话框
        this.tempGroup.id = this.list[$index].id;
        this.tempGroup.uid = this.list[$index].uid;
        this.tempGroup.name = this.list[$index].name;
        this.dialogStatus = "update"
        this.dialogFormVisible = true
      },
      createGroup() {
        //保存新服务器
        this.api({
          url: "/group/addGroup",
          method: "post",
          data: this.tempGroup
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      updateGroup() {
        //修改服务器
        this.api({
          url: "/group/updateGroup",
          method: "post",
          data: this.tempGroup
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      getUserList() {
        this.api({
          url: "/user/list",
          method: "get"
        }).then(data => {
          this.userList = data.list;
        })
      },
      delGroup($index) {
        this.$confirm('此操作将删除该组空间，组空间下资源将被删除, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.api({
            url: "/group/delGroup",
            method: "post",
            data: {id: this.list[$index].id}
          }).then(data => {
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
            this.getList();
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
      }
    }
  }
</script>
