<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <el-table :data="rolesList" border stripe>
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', index1 === 0? 'bdtop' : '','vcenter']" v-for="(item1,index1) in scope.row.children" :key="item1.id">
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <el-col :span="19">
                            <el-row :class="[index2 === 0? '': 'bdtop', 'vcenter']" v-for="(item2,index2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{ item3.authName }}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleDialogClosed">
        <el-form status-icon :rules="addRoleFormRules" :model="addRoleForm" ref="addRoleFormRef" label-width="80px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="addRoleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="addRoleForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="50%" @close="editRoleDialogClosed">
        <el-form status-icon :model="editRoleForm" ref="editRoleFormRef" label-width="80px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="editRoleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="editRoleForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRole">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <el-tree ref="treeRef" :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="setRight">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    name: 'Roles',
    data() {
        return {
            rolesList: [],
            rightsList: [],
            roleId: '',
            addRoleDialogVisible: false,
            editRoleDialogVisible: false,
            setRightDialogVisible: false,
            addRoleForm: {
                roleName: '',
                roleDesc: ''
            },
            editRoleForm: {},
            addRoleFormRules: {
                roleName: [{
                    required: true,
                    message: '请输入角色名',
                    trigger: 'blur'
                }],
                roleDesc: [{
                    required: false,
                    message: '请输入角色描述',
                    trigger: 'blur'
                }]
            },
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            defKeys: []
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const {
                data: res
            } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败！')
            }
            this.rolesList = res.data
        },
        addRoleDialogClosed() {
            this.$refs.addRoleFormRef.resetFields()
        },
        editRoleDialogClosed() {
            this.$refs.editRoleFormRef.resetFields()
        },
        addRole() {
            this.$refs.addRoleFormRef.validate(async vaild => {
                if (!vaild) return
                const {
                    data: res
                } = await this.$http.post('roles', this.addRoleForm)
                if (res.meta.status !== 201) {
                    return this.$message.error('创建角色失败')
                }
                this.addRoleDialogVisible = false
                this.$message.success('创建角色成功！')
                this.getRolesList()
            })
        },
        async showEditRoleDialog(id) {
            const {
                data: res
            } = await this.$http.get('roles/' + id)
            if (res.meta.status !== 200) {
                this.$message.error("查询用户信息失败！")
            }
            this.editRoleForm = res.data
            this.editRoleDialogVisible = true
            console.log(this.editRoleForm)
        },
        editRole() {
            this.$refs.editRoleFormRef.validate(async vaild => {
                if (!vaild) return
                const {
                    data: res
                } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
                    roleName: this.editRoleForm.roleName,
                    roleDesc: this.editRoleForm.roleDesc
                })
                if (res.meta.status !== 200) {
                    return this.$message.error('修改角色失败')
                }
                this.editRoleDialogVisible = false
                this.$message.success('创建角色成功！')
                this.getRolesList()
            })
        },
        async deleteRole(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {
                data: res
            } = await this.$http.delete('roles/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除角色失败！')
            }
            this.$message.success('删除角色成功！')
            this.getRolesList()
        },
        async removeRightById(role, rightId) {
            const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {
                data: res
            } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if (res.meta.status !== 200) {
                return this.$message.error('删除权限失败！')
            }
            role.children = res.data
            this.$message.success('删除权限成功！')
        },
        async showSetRightDialog(role) {
            this.roleId = role.id
            const {
                data: res
            } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取用户权限失败！')
            }
            this.rightsList = res.data
            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true
        },
        getLeafKeys(node, arr) {
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafKeys(item, arr))
        },
        setRightDialogClosed() {
            this.defKeys = []
        },
        async setRight() {
            const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
            const idStr = keys.join(',')
            console.log(idStr)
            const {
                data: res
            } = await this.$http.post(`roles/${this.roleId}/rights`, {
                rids: idStr
            })
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败！')
            }
            this.$message.success('分配权限成功！')
            this.getRolesList()
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
