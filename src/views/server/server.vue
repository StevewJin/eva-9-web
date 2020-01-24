<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form>
        <el-form-item>
          <el-button type="primary" icon="plus" @click="showCreate" v-if="hasPerm('server:add')">添加
          </el-button>
        </el-form-item>
      </el-form>
    </div>
    <el-table :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit
              highlight-current-row>
      <el-table-column align="center" label="序号" width="80">
        <template slot-scope="scope">
          <span v-text="getIndex(scope.$index)"> </span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="name" label="名称" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="ip" label="IP" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="username" label="用户名" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="remark" label="备注" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="创建时间" width="170">
        <template slot-scope="scope">
          <span>{{scope.row.createTime}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="createBy" label="创建人" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="管理" width="200" v-if="hasPerm('article:update')">
        <template slot-scope="scope">
          <el-button type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button type="primary" icon="connect" @click="connectServer(scope.$index)">连接</el-button>
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
      <el-form class="small-space" :model="tempServer" label-position="left" label-width="120px"
               style='width: 400px; margin-left:100px;'>
        <el-form-item label="服务器名称">
          <el-input type="text" v-model="tempServer.name">
          </el-input>
        </el-form-item>
        <el-form-item label="IP">
          <el-input type="text" v-model="tempServer.ip">
          </el-input>
        </el-form-item>
        <el-form-item label="用户名">
          <el-input type="text" v-model="tempServer.username">
          </el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input type="text" v-model="tempServer.password">
          </el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input type="text" v-model="tempServer.remark">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createArticle">创 建</el-button>
        <el-button type="primary" v-else @click="updateArticle">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data() {
      return {
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
        tempServer: {
          id: "",
          name: "",
          ip: "",
          username: "",
          password: "",
          remark: ""
        }
      }
    },
    created() {
      this.getList();
    },
    methods: {
      getList() {
        //查询列表
        if (!this.hasPerm('server:list')) {
          return
        }
        this.listLoading = true;
        this.api({
          url: "/server/listServer",
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
        this.tempServer.name = "";
        this.tempServer.ip = "";
        this.tempServer.username = "";
        this.tempServer.password = "";
        this.tempServer.remark = "";
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
      showUpdate($index) {
        //显示修改对话框
        this.tempServer.id = this.list[$index].id;
        this.tempServer.name = this.list[$index].name;
        this.tempServer.ip = this.list[$index].ip;
        this.tempServer.username = this.list[$index].username;
        this.tempServer.password = this.list[$index].password;
        this.tempServer.remark = this.list[$index].remark;
        this.dialogStatus = "update"
        this.dialogFormVisible = true
      },
      createArticle() {
        //保存新服务器
        this.api({
          url: "/server/addServer",
          method: "post",
          data: this.tempServer
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      updateArticle() {
        //修改服务器
        this.api({
          url: "/server/updateServer",
          method: "post",
          data: this.tempServer
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      connectServer($index) {
        //连接服务器
        this.tempServer.id = this.list[$index].id;
        this.tempServer.name = this.list[$index].name;
        this.tempServer.ip = this.list[$index].ip;
        this.tempServer.username = this.list[$index].username;
        this.tempServer.password = this.list[$index].password;
        this.tempServer.remark = this.list[$index].remark;
        this.api({
          url: "/server/connectServer",
          method: "post",
          data: this.tempServer
        }).then((data) => {
          
        })
      }
    }
  }
</script>
