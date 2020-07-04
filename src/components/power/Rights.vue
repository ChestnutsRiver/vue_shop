<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <div slot="header">
        <el-table
          :data="rightsList"
          style="width: 100%"
          border
          stripe>
          <el-table-column
            type="index"
            label="#" >
          </el-table-column>
          <el-table-column
            prop="authName"
            label="权限名称" >
          </el-table-column>
          <el-table-column
            prop="path"
            label="路径" >
          </el-table-column>
          <el-table-column
            prop="level"
            label="权限等级" >
            <template v-slot="scope">
              <el-tag v-if="scope.row.level === '0'">一级</el-tag>
              <el-tag v-else-if="scope.row.level === '1'" type="success">二级</el-tag>
              <el-tag v-else type="warning">三级</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div></div>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rightsList: []
    }
  },
  created() {
    this.getRightsList()
  },
  methods: {
    getRightsList() {
      this.$http.get('rights/list').then(res => {
        if (res.data.meta.status !== 200) {
          return this.$message.error('获得权限列表失败！')
        }
        this.rightsList = res.data.data
      })
    }
  }
}
</script>

<style>

</style>
