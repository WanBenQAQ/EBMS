<template>
  <div>
    <!--面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <!--添加角色按钮区域-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleVisble = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table :data="rolelist" border stripe>
        <!--展开列-->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', index1 === 0 ? 'bdtop' : '', 'vcenter']"
                    v-for="(item1, index1) in scope.row.children" :key="item1.id">
              <!--渲染一级权限-->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--渲染二级和三级权限-->
              <el-col :span="19">
                <!--通过for循环 嵌套渲染二级权限-->
                <el-row :class="[index2 === 0 ? '' : 'bdtop', 'vcenter']"
                        v-for="(item2, index2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable
                            @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning"
                            v-for="item3 in item2.children"
                            :key="item3.id" closable
                            @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!--索引列-->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRole(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="waring" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!--添加用户的对话框-->
    <el-dialog title="添加角色" :visible.sync="addRoleVisble"
               width="50%" @close="addRoleClosed">
      <!--内容主体区域-->
      <el-form :model="addForm" :rules="addFormRules"
               ref="addFormRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区域-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisble = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改角色的对话框-->
    <el-dialog
      title="修改角色" :visible.sync="editRoleVisible" width="50%" @close="editRoleClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!--分配权限的对话框-->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!--树形控件-->
      <el-tree :data="rightslist" :props="treeProps"
               show-checkbox node-key="id"
               default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "Roles",
    data() {
      return {
        // 所有角色列表数据
        rolelist: [],
        // 控制添加角色对话框的显示与隐藏
        addRoleVisble: false,
        // 添加角色的表单数据
        addForm: {
          roleName: '',
          roleDesc: '',
        },
        addFormRules: {
          roleName: [
            { required: true, message: '请输入角色名', trigger: 'blur' }
          ],
          roleDesc: [
            { required: true, message: '请输入描述内容', trigger: 'blur' }
          ]
        },
        // 控制修改角色对话框的显示与隐藏
        editRoleVisible: false,
        // 查询到的角色信息对象
        editForm: {},
        editFormRules: {
          roleName: [
            { required: true, message: '请输入角色名', trigger: 'blur' }
          ],
          roleDesc: [
            { required: true, message: '请输入描述内容', trigger: 'blur' }
          ]
        },
        // 控制分配权限对话框的显示与隐藏
        setRightDialogVisible: false,
        // 所有权限的数据
        rightslist: [],
        // 树形控件的属性绑定对象
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        // mo'ren选中的节点Id值数组
        defKeys: [],
        // 当前即将分配权限的角色id
        roleId: ''
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      // 获取所有角色的列表
      async getRolesList() {
        const {data: res} = await this.$http.get('roles')
        if(res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败!')
        }
        this.rolelist = res.data
      },
      // 监听添加角色对话框的关闭事件
      addRoleClosed() {
        // resetFields:重置字段
        this.$refs.addFormRef.resetFields()
      },
      // 添加角色
      addRole() {
        this.$refs.addFormRef.validate(valid => {
          if(!valid) return this.$message.error('添加失败，请补全角色信息')
          // 可以发起添加角色的网络请求
          this.$http.post('roles', this.addForm).then(res => {
            if(res.data.meta.status !== 201) {
              return this.$message.error('角色创建失败!')
            }
            this.$message.success('角色添加成功!')
            // 隐藏添加用户的对话框
            this.addRoleVisble = false
            // 重新获取用户列表数据
            this.getRolesList()
          })
        })
      },
      // 展示编辑角色的对话框
      showEditRole(id) {
        this.$http.get('roles/' + id).then(res => {
          if(res.data.meta.status !== 200) {
            return this.$message.error('查询角色信息失败!')
          }
          this.editForm = res.data.data
        })
        this.editRoleVisible = true
      },
      // 监听修改角色对话框的关闭事件
      editRoleClosed() {
        // resetFields:重置字段
        this.$refs.editFormRef.resetFields()
      },
      // 修改角色信息并提交
      editRoleInfo() {
        this.$refs.editFormRef.validate(valid => {
          if(!valid) return this.$message.error('未通过验证')
          // 发起修改角色信息的数据请求
          this.$http.put('roles/' + this.editForm.roleId, {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          }).then(res => {
            if(res.data.meta.status !== 200) return this.$message.error('更新角色信息失败')
            // 关闭对话框
            this.editRoleVisible = false
            // 刷新数据列表
            this.getRolesList()
            // 提示修改成功
            this.$message.success('更新角色信息成功')
          })
        })
      },
      // 根据Id删除对应的角色信息
      removeRoleById(id) {
        // 弹框询问用户是否删除数据
        this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(res => {
          this.$http.delete('roles/' + id).then(res => {
            if(res.data.meta.status !== 200) return this.$message.error('删除角色失败!')
            this.$message.success('删除角色成功!')
            // 刷新数据列表
            this.getRolesList()
          })
        }).catch(() => {
          return this.$message.info('取消本次删除')
        })
      },
      // 根据Id删除对应的权限
      async removeRightById(role, rightId) {
        // 弹窗提示用户是否要删除
        const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') {
          return this.$message.info('取消删除')
        }
        const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !== 200) {
          return this.$message.error('删除权限失败')
        }
        this.$message.success('删除权限成功')
        role.children = res.data
      },
      // 展示分配权限的对话框
      async showSetRightDialog(role) {
        // 点击展示分配权限时获取对应角色的id，方便后续给对应角色分配权限
        this.roleId = role.id
        // 获取所有权限的数据
        const {data: res} = await this.$http.get('rights/tree')
        if(res.meta.status !== 200) {
          return this.$message.error('获取权限数据失败')
        }
        // 把获取到的权限数据保存到 data 中
        this.rightslist = res.data
        // 递归获取三级节点的Id
        this.getLeafKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      },
      // 通过递归的形式，获取角色下所有三级权限的id，并保存到数组中
      getLeafKeys(node, arr) {
        // 如果当前 mode 节点不包含 children 属性，则是三级节点
        if(!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(item => {
          this.getLeafKeys(item, arr)
        })
      },
      // 监听分配权限对话框的关闭事件
      setRightDialogClosed() {
        this.defKeys = []
      },
      // 点击为角色分配权限
      async allotRights() {
        console.log('---');
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        const idStr = keys.join(',')
        const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
        if(res.meta.status !== 200) {
          this.$message.error('分配权限失败')
        }
        this.$message.success('分配权限成功')
        // 重新刷新列表
        this.getRolesList()
        // 关闭对话框
        this.setRightDialogVisible = false
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
