<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片部分 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRolesDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['vcenter','bdbottom', index1 === 0 ? 'bdtop' : 'bdbottom']"
              v-for="(item1,index1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="6">
                <el-tag closable @close="closedThreeRights(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="18">
                <el-row
                  :class="['vcenter',index2 === 0 ? '' : 'bdtop']"
                  v-for="(item2,index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      closable
                      @close="closedThreeRights(scope.row,item2.id)"
                      type="success"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="closedThreeRights(scope.row,item3.id)"
                      type="warning"
                      :class="[index3 === 0 ? '' : 'bdtop']"
                      v-for="(item3,index3) in item2.children"
                      :key="item3.id"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="editChoseRolesForm(scope.row.id)"
            >编辑</el-button>
            <el-button
              @click="editDelete(scope.row.id)"
              type="danger"
              icon="el-icon-delete"
              size="mini"
            >删除</el-button>
            <el-button
              type="warning"
              @click="showDialogAddRights(scope.row)"
              icon="el-icon-setting"
              size="mini"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog
      @close="addRolesClosed"
      title="添加角色"
      :visible.sync="addRolesDialogVisible"
      width="50%"
    >
      <el-form :model="addRolesForm" :rules="addRolesRules" ref="addRolesRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色的对话框 -->
    <el-dialog
      @close="editRolesClosed"
      title="编辑角色"
      :visible.sync="editRolesDialogVisible"
      width="50%"
    >
      <el-form :model="editRolesForm" :rules="addRolesRules" ref="editRolesRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog title="提示" :visible.sync="setRightsDialogVisible" width="50%">
      <!-- 树形 -->
      <el-tree :data="rightsList" ref="treeRef" :props="treetProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      //   添加角色模块
      addRolesDialogVisible: false,
      addRolesForm: {
        roleName: '',
        roleDesc: ''
      },
      addRolesRules: {
        roleName: [
          {
            required: true,
            message: '请输入角色名称',
            tigger: 'blur'
          }
        ],
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            tigger: 'blur'
          }
        ]
      },
      //   编辑角色模块
      editRolesDialogVisible: false,
      editRolesForm: {},
      // 控制分配权限的对话框
      setRightsDialogVisible:false,
      rightsList:[],
      // 树形控件
      treetProps:{
        children:'children',
        label:'authName'
      },
      defKeys:[],
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('角色列表获取失败')
      }
      // console.log(res.data)
      this.roleList = res.data
    },
    // 监听添加用户框关闭
    addRolesClosed() {
      this.$refs.addRolesRef.resetFields()
    },
    // 点击确认之后先进行表单预校验,然后发送请求
    addRoles() {
      this.$refs.addRolesRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('创建角色失败')
        }
        this.$message.success('创建角色成功')
        this.addRolesDialogVisible = false
        this.getRolesList()
      })
    },
    // 获取该角色的数据
    async editChoseRolesForm(id) {
      this.editRolesDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色失败')
      }
      this.editRolesForm = res.data
      // console.log(this.editRolesForm)
    },
    editRolesClosed() {
      this.$refs.editRolesRef.resetFields()
    },
    editRoles() {
      this.$refs.editRolesRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editRolesForm.roleId,
          {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色失败')
        }
        this.editRolesDialogVisible = false
        this.$message.success('修改用户成功')
        this.getRolesList()
      })
    },
    async editDelete(id) {
      const editDeleteMess = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '删除提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      //   console.log(editDeleteMess)
      if (editDeleteMess !== 'confirm') {
        return this.$message.info('取消了删除操作')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getRolesList()
    },
    // 删除权限的操作
    async closedThreeRights(roledata, rightId) {
      const removeMess = await this.$confirm(
        '此操作将永久删除该角色权限, 是否继续?',
        '删除提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (removeMess !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      // console.log(roledata)
      // console.log(rightId)
      const { data: res } = await this.$http.delete(
        `roles/${roledata.id}/rights/${rightId}`
      )
      // this.$message.warning('删除')
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // this.getRolesList()
      roledata.children = res.data
    },
    // 展示分配权限的弹框
    async showDialogAddRights(role) {
      this.roleId = role.id
      const {data:res} = await this.$http.get('rights/tree');
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      this.rightsList = res.data;
      // s.log(role);
      // 递归寻找三级节点
      this.defKeys=[]
      this.getLeafKeys(role,this.defKeys)
      this.setRightsDialogVisible = true;
    },
    getLeafKeys(node, arr){
      // 判断三级接点,不包含则
      if(!node.children){
        return arr.push(node.id)
      }
      // 递归
      node.children.forEach(item => {
        this.getLeafKeys(item,arr)
      })
      // console.log(arr)
    },
    async setRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ];
      const idStr = keys.join(',');
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status !== 200){
        return this.$message.error('分配权限失败')
      }
      this.setRightsDialogVisible = false;
      this.$message.success('分配权限成功')
      this.getRolesList()
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>