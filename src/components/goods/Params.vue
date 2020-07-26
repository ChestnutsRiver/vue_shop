<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 级联选择器 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数按钮 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
          >添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <!-- 循环渲染tag标签 -->
              <template v-slot="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, scope.row)"
                >{{ item }}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(socpe.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column lable="#" type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParamById(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性按钮 -->
          <el-button
            type="
          primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
          >添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <!-- 循环渲染tag标签 -->
              <template v-slot="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, scope.row)"
                >{{ item }}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(socpe.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" lable="#"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParamById(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="40%"
      @close="addDialogClosed"
    >
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button @click="addDialogClosed">重 置</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="40%"
      @close="editDialogClosed"
    >
      <el-form ref="editFormRef" :model="editForm" :rules="addFormRules" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button @click="editDialogClosed">重 置</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类赋值列表
      cateList: [],
      // 级联选择器的配置对象
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框双向绑定的数组
      selectedCateKeys: [],
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态属性数据
      onlyTableData: [],
      addDialogVisible: false,
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      // 编辑参数的表单数据对象
      editForm: {
        attr_name: ''
      }
      // 控制按钮与文本框的切换显示
      // inputVisible: false,
      // 文本框输入值
      // inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    getCateList() {
      this.$http.get('categories').then(res => {
        if (res.data.meta.status !== 200) {
          return this.$message.error('获得商品分类失败！')
        }
        this.cateList = res.data.data
      })
    },
    // 监听级联选择框数据变化,获得参数值
    handleChange() {
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      this.$http
        .get(`categories/${this.cateId}/attributes`, {
          params: { sel: this.activeName }
        })
        .then(res => {
          if (res.data.meta.status !== 200) {
            return this.$message.error('获得参数列表失败！')
          }
          // console.log(res.data.data)
          res.data.data.forEach(item => {
            item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
            item.inputVisible = false
            item.inputValue = ''
          })

          if (this.activeName === 'many') {
            this.manyTableData = res.data.data
          } else {
            this.onlyTableData = res.data.data
          }
        })
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      // console.log(this.activeName)
      this.handleChange()
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数
    addParams() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return
        this.$http
          .post(`categories/${this.cateId}/attributes`, {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })
          .then(res => {
            if (res.data.meta.status !== 201) {
              return this.$message.error('添加参数失败！')
            }
            this.$message.success('添加参数成功！')
            this.addDialogVisible = false
            this.handleChange()
          })
      })
    },
    // 编辑对话框
    showEditDialog(attrId) {
      this.$http
        .get(`categories/${this.cateId}/attributes/${attrId}`, {
          params: { attr_sel: this.activeName }
        })
        .then(res => {
          if (res.data.meta.status !== 200) {
            return this.$message.error('添加参数信息败！')
          }
          this.editForm = res.data.data
          this.editDialogVisible = true
        })
    },
    // 提交编辑参数
    editParams() {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$http
          .put(
            `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
            { attr_sel: this.activeName, attr_name: this.editForm.attr_name }
          )
          .then(res => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('修改参数信息败！')
            }
            this.$message.success('修改参数信息成功')
            this.handleChange()
            this.editDialogVisible = false
          })
      })
    },
    // 重置修改对话框
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    removeParamById(id) {
      // 弹框询问用户是否删除数据
      this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http
            .delete(`categories/${this.cateId}/attributes/${id}`)
            .then(res => {
              if (res.data.meta.status !== 200) {
                return this.$message.error('删除参数失败!')
              }
              this.$message.success('删除参数成功!')
              // console.log(cate)
              this.handleChange()
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 文本框失去焦点，或者摁下enter都会触发
    handleInputConfirm(row) {
      // console.log('ok')
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果没有return, 则证明输入的内容需要做后续处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发起请求，保存这次操作
      this.saveAttr(row)
    },
    saveAttr(row) {
      this.$http
        .put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(',')
        })
        .then(res => {
          if (res.data.meta.status !== 200) {
            return this.$message.error('修改参数失败!')
          }
          this.$message.success('修改参数成功!')
        })
    },
    // 点击按钮，展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // $nextTick 方法的作用，当页面上元素重新渲染之后，才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttr(row)
    }
  },
  computed: {
    // 判断按钮是否需要禁用
    isBtnDisabled() {
      return this.selectedCateKeys.length !== 3
    },
    // 当前选中的三级分类
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 参数名称
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin-top: 15px;
}

.el-cascader {
  width: 300px;
}

.el-tag {
  margin: 10px;
}

.input-new-tag {
  width: 120px;
}
</style>
