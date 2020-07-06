<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              :class="['bdbottom', i1 === 0? 'bdtop': '', 'vcenter']"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0? '': 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >{{ item3.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>{{ scope.row }}</pre> -->
          </template>
        </el-table-column>
        <!-- 索引列  -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoleDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配角色</el-button>
            <!-- {{scope.row}} -->
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="45%"
      @close="addRoleDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="addRoleForm"
        :rules="addRoleFormRules"
        ref="addRoleFormRef"
        label-width="90px"
        status-icon
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button @click="addRoleDialogClosed">重 置</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editRoleDialogVisible"
      width="45%"
      @close="editRoleDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="editRoleForm"
        :rules="addRoleFormRules"
        ref="editRoleFormRef"
        label-width="90px"
        status-icon
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button @click="editRoleDialogClosed">重 置</el-button>
        <el-button type="primary" @click="modifyRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="setRightdialogVisible" width="35%">
      <!-- 树形控件 -->
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      addRoleDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      addRoleFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      editRoleDialogVisible: false,
      editRoleForm: {},
      setRightdialogVisible: false,
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点Id数据值
      defKeys: [],
      // 当前分配角色的Id值
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表数据
    getRolesList() {
      this.$http.get('roles').then(res => {
        // console.log(res)
        if (res.data.meta.status !== 200) {
          return this.$message.error('获取角色列表失败！')
        }
        this.roleList = res.data.data
      })
    },
    // 重置添加用户对话框
    addRoleDialogClosed() {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 添加用户
    addRole() {
      this.$refs.addRoleFormRef.validate(valid => {
        if (!valid) return
        // 发起添加角色请求
        this.$http.post('roles', this.addRoleForm).then(res => {
          if (res.data.meta.status !== 201) {
            return this.$message.error('添加用户角色失败！')
          }
          this.$message.success('添加用户角色成功')
          // 隐藏对话框
          this.addRoleDialogVisible = false
          // 重新获取角色列表数据
          this.getRolesList()
        })
      })
    },
    // 重置编辑用户对话框
    editRoleDialogClosed() {
      this.$refs.editRoleFormRef.resetFields()
    },
    // 展示修改用户对话框
    showEditRoleDialog(id) {
      this.$http.get('roles/' + id).then(res => {
        if (res.data.meta.status !== 200) {
          return this.$message.error('查询角色信息失败')
        }
        this.editRoleForm = res.data.data
        this.editRoleDialogVisible = true
      })
    },
    // 修改角色信息并提交
    modifyRoleInfo() {
      this.$refs.editRoleFormRef.validate(valid => {
        if (!valid) return
        // console.log(this.editRoleForm)
        // 发起修改角色请求
        this.$http
          .put('roles/' + this.editRoleForm.roleId, {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc
          })
          .then(res => {
            // console.log(res)
            if (res.data.meta.status !== 200) {
              return this.$message.error('修改角色失败！')
            }
            this.$message.success('修改角色成功')
            // 隐藏对话框
            this.editRoleDialogVisible = false
            // 重新获取角色列表数据
            this.getRolesList()
          })
      })
    },
    // 删除角色
    removeRoleById(id) {
      // 弹框询问用户是否删除数据
      this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http.delete('roles/' + id).then(res => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('删除角色失败!')
            }
            this.$message.success('删除角色成功!')
            this.getRolesList()
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 根据Id删除对应权限
    removeRightById(role, rightId) {
      // 弹框询问用户是否删除数据
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http.delete(`roles/${role.id}/rights/${rightId}`).then(res => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('删除权限失败!')
            }
            this.$message.success('删除权限成功!')
            // this.getRolesList()
            // 上面会重新刷新页面 采用下面
            role.children = res.data.data
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 展示分配权限对话框
    showSetRightDialog(role) {
      this.roleId = role.id
      this.$http.get('rights/tree').then(res => {
        if (res.data.meta.status !== 200) {
          return this.$message.error('获得权限列表失败！')
        }
        this.rightsList = res.data.data
        // console.log(this.rightsList)
        // 每次打开前将defKeys清空
        this.defKeys = []
        this.getLeafKeys(role, this.defKeys)
        this.setRightdialogVisible = true
      })
    },
    // 通过递归获取角色下所有三级权限的Id，并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 点击为角色分配权限
    allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')
      this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr }).then(res => {
        if (res.data.meta.status !== 200) {
          return this.$message.error('分配权限失败!')
        }
        this.$message.success('分配权限成功!')
        this.getRolesList()
        this.setRightdialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
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
