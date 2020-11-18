<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            </el-col>
        </el-row>
        <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border>
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
                <i class="el-icon-error" v-else style="color: red"></i>
            </template>
            <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
                <el-tag size="mini" v-else-if="scope.row.cat_level===1" type="success">二级</el-tag>
                <el-tag size="mini" v-else type="warning">三级</el-tag>
            </template>
            <template slot="opt" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCateDialog(scope.row)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCate(scope.row)">删除 </el-button>
            </template>
        </tree-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
        <el-form status-icon :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
            <el-form-item label="分类名称：" prop="cat_name">
                <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类：">
                <el-cascader clearable v-model="selectedKeys" :options="parentCateList" :props="cascaderProps" @change="parentCateChange" change-on-select></el-cascader>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="编辑分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
        <el-form status-icon :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
            <el-form-item label="分类名称：" prop="cat_name">
                <el-input v-model="editCateForm.cat_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editCate">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    name: 'Cate',
    data() {
        return {
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            editCateForm: {},
            addCateDialogVisible: false,
            editCateDialogVisible: false,
            total: 0,
            catelist: [],
            columns: [{
                    label: '分类名称',
                    prop: 'cat_name'
                },
                {
                    label: '是否有效',
                    type: 'template',
                    template: 'isok'
                },
                {
                    label: '排序',
                    type: 'template',
                    template: 'order'
                },
                {
                    label: '操作',
                    type: 'template',
                    template: 'opt'
                }
            ],
            addCateFormRules: {
                cat_name: [{
                    required: true,
                    message: '请输入分类名称',
                    trigger: 'blur'
                }]
            },
            editCateFormRules: {
                cat_name: [{
                    required: true,
                    message: '请输入分类名称',
                    trigger: 'blur'
                }]
            },
            parentCateList: [],
            cascaderProps: {
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedKeys: []
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const {
                data: res
            } = await this.$http.get('categories', {
                params: this.queryInfo
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品信息失败！')
            }
            this.catelist = res.data.result
            this.total = res.data.total
        },
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getCateList()
        },
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getCateList()
        },
        async getParentCateList() {
            const {
                data: res
            } = await this.$http.get('categories', {
                params: {
                    type: 2
                }
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取父级数据失败！')
            }
            this.parentCateList = res.data
        },
        showAddCateDialog() {
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        parentCateChange() {
            if (this.selectedKeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                this.addCateForm.cat_level = this.selectedKeys.length
                return
            } else {
                this.addCateForm.cat_pid = 0
                this.addCateForm.cat_level = 0
            }
        },
        addCate() {
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return
                const {
                    data: res
                } = await this.$http.post('categories', this.addCateForm)
                if (res.meta.status !== 201) {
                    return this.$message.error('添加分类失败！')
                }
                this.$message.success('添加分类成功！')
                this.getCateList()
                this.addCateDialogVisible = false
            })
        },
        addCateDialogClosed() {
            this.$refs.addCateFormRef.resetFields()
            this.selectedKeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0
        },
        editCateDialogClosed() {
            this.$refs.editCateFormRef.resetFields()
        },
        async showEditCateDialog(cate) {
            const {
                data: res
            } = await this.$http.get('categories/' + cate.cat_id)
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品类型失败！')
            }
            this.editCateForm = res.data
            this.editCateDialogVisible = true
        },
        async editCate() {
            const {
                data: res
            } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
                cat_name: this.editCateForm.cat_name
            })
            if (res.meta.status !== 200) {
                return this.$message.error('修改信息失败！')
            }
            this.getCateList()
            this.$message.success('修改信息成功！')
            this.editCateDialogVisible = false
        },
        async deleteCate(cate) {
            const confirmResult = await this.$confirm('此操作将永久删除该商品类型, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {
                data: res
            } = await this.$http.delete('categories/' + cate.cat_id)
            if (res.meta.status !== 200) {
                return this.message.error('删除商品类型失败！')
            }
            this.getCateList()
            this.$message.success('删除商品类型成功！')
        }
    }
}
</script>

<style lang="less" scoped>
.treeTable {
    margin: 15px 0;
}

.el-cascader {
    width: 100%;
}
</style>
