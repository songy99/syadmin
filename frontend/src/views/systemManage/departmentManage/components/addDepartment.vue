<template>
    <el-dialog
            :title="dialogTitle"
            :visible.sync="dialogVisible"
            width="640px"
            center
            :destroy-on-close="true"
            :close-on-click-modal="false"
            :before-close="handleClose">
        <el-form :inline="true" :model="formData" :rules="rules" ref="rulesForm" label-position="right" label-width="130px" class="form-store">
            <el-form-item label="ID：" prop="id" v-if="dialogTitle=='详情'">
                <el-input v-model.trim="formData.id" disabled style="width: 380px"></el-input>
            </el-form-item>
            <el-form-item label="父级部门：" prop="parent">
                <el-cascader
                        style="width: 300px"
                        :show-all-levels="false"
                        :options="options"
                        ref="myCascader"
                        v-model="formData.parent"
                        @change="handleChange"
                        :props="{ checkStrictly: true ,label:'name',value:'id'}"
                        clearable></el-cascader>
            </el-form-item>
            <el-form-item label="部门名称：" prop="name">
                <el-input v-model.trim="formData.name" style="width: 380px"></el-input>
            </el-form-item>
            <el-form-item label="负责人：" prop="owner">
                <el-input v-model.trim="formData.owner" style="width: 380px"></el-input>
            </el-form-item>
            <el-form-item label="联系电话：" prop="phone">
                <el-input v-model.trim="formData.phone" style="width: 380px"></el-input>
            </el-form-item>
            <el-form-item label="邮箱：" prop="email">
                <el-input v-model.trim="formData.email" style="width: 380px"></el-input>
            </el-form-item>
            <el-form-item label="状态：" prop="status">
                <el-radio-group v-model="formData.status" style="width: 380px">
                    <el-radio :label="1">启用</el-radio>
                    <el-radio :label="0">禁用</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item label="排序：" prop="sort">
                <el-input-number v-model="formData.sort" :min="1" :max="999999"></el-input-number>
            </el-form-item>

        </el-form>
        <span slot="footer">
            <el-button @click="handleClose" :loading="loadingSave">取消</el-button>
            <el-button type="primary" @click="submitData" :loading="loadingSave" v-if="dialogTitle!='详情'">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
    import utils from '@/utils/util'
    import {apiSystemDeptAdd,apiSystemDept,apiSystemDeptEdit} from '@/api/api'
    export default {
        name: "addDepartment",
        data() {
            return {
                dialogVisible:false,
                loadingSave:false,
                dialogTitle:'',
                formData:{
                    id:null,
                    parent:'',
                    name:'',
                    phone:'',
                    owner:'',
                    status:1,
                    sort:'',
                },
                rules:{
                    // parent: [
                    //     {required: true, message: '请选择父级菜单',trigger: 'blur'}
                    // ],
                    name: [
                        {required: true, message: '请输入部门名称',trigger: 'blur'}
                    ],
                    // phone: [
                    //     {required: true, message: '请输入联系电话',trigger: 'blur'}
                    // ],
                    // owner: [
                    //     {required: true, message: '请输入负责人',trigger: 'blur'}
                    // ],
                    status: [
                        {required: true, message: '请选择状态',trigger: 'blur'}
                    ],
                    // sort: [
                    //     {required: true, message: '请输入排序',trigger: 'blur'}
                    // ],
                },
                options: []
            }
        },
        methods:{
            handleChange(e){
                 //console.log(e,'e---- 查看根节点')
                 //console.log(this.$refs.myCascader.getCheckedNodes()[0].pathLabels )
                 //console.log(this.$refs.myCascader.getCheckedNodes()[0].value )
                  // var label = this.$refs.myCascader.getCheckedNodes()[0].label
                 this.formData.parent = this.$refs.myCascader.getCheckedNodes()[0].value
                 if (this.formData.parent == "undefined"){
                     this.formData.parent =""
                 }
            },
            handleClose() {
                this.dialogVisible=false
                this.loadingSave=false
            },
            addDepartmentFn(item,flag) {
                this.dialogVisible=true
                this.dialogTitle=flag
                this.formData=item ? item : {
                    id:null,
                    parent:'',
                    name:'',
                    phone:'',
                    owner:'',
                    status:1,
                    sort:'',
                }
                this.getapiSystemDept()
            },
            submitData() {
                //console.log(typeof this.formData.parent , 'formData.parentbbbbb 888')
                let param = {
                    ...this.formData
                }
                // if(typeof this.formData.parent== 'object') {
                //     param.parent = param.parent[param.parent-1]
                // }
                this.$refs['rulesForm'].validate(obj=>{
                    if(obj) {
                        this.loadingSave=true
                        //console.log(this.formData,'this.formData----')
                        if(this.dialogTitle=="新增"){
                            apiSystemDeptAdd(param).then(res=>{
                                this.loadingSave=false
                                if(res.code == 2000) {
                                    this.$message.success(res.msg)
                                    this.handleClose()
                                    this.$emit('refreshData')
                                } else {
                                    this.$message.warning(res.msg)
                                }
                            })
                        }else{
                            apiSystemDeptEdit(this.formData).then(res=>{
                                this.loadingSave=false
                                if(res.code == 2000) {
                                    this.$message.success(res.msg)
                                    this.handleClose()
                                    this.$emit('refreshData')
                                } else {
                                    this.$message.warning(res.msg)
                                }
                            })
                        }

                    }
                })
            },
            getapiSystemDept() {
                apiSystemDept().then(res=>{
                    if(res.code == 2000) {
                        let childrenList = res.data.data.filter(item=> item.parent)
                        let parentList = res.data.data.filter(item=> !item.parent)
                        if(parentList.length >0) {
                            parentList.forEach(item=>{
                                let children = childrenList.filter(itema=>itema.parent == item.id)
                                item.children=[...children]
                            })

                            let menu = [{
                                label:'-1',
                                name:'根节点',
                                children:[...parentList]
                            }]
                            this.options = menu
                        }

                    } else {
                        this.$message.warning(res.msg)
                    }
                })
            }
        }
    }
</script>

