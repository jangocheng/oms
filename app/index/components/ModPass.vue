<template lang="html">
    <section class="L-background">
        <div class="L-width">
            <el-row :gutter="10">
                <!-- 上传照片 -->
                <el-col :span="5" :offset="2">
                    <div class="L-grid-content L-bg-purple">
                       <el-upload class="avatar-uploader" :action="imgurl" :data="this.addForm" :show-file-list="false" name="userPhoto"
                        :on-success="handleAvatarScucess" :before-upload="beforeAvatarUpload" :headers="{Token: this.cookie}">
                           <img v-if="imageUrl" :src="imageUrl" class="avatar">
                           <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                       </el-upload>
                    </div>
                </el-col>
                <el-col :span="5">
                    <div class="L-grid-content L-bg-purple">
                        <el-form>
                            <el-form-item class="L-form-item" :label="item" v-for="item in menu" :key="item">
                            </el-form-item>
                        </el-form>
                    </div>
                </el-col>
                <!-- 个人信息 -->
                <el-col :span="10">
                    <div class="L-grid-content L-bg-purple">
                        <el-form :model="addForm" :rules="userEditFormRules">
                              <el-form-item class="L-form-item" prop="name">
                                  <el-input :disabled="!name" v-model="addForm.userName" class="L-input">
                                      <el-button slot="append" class="L-input-button" @click.stop.native="changeName">
                                          <img :src="changeNameImgUrl" :alt="statesName" :title="statesName">
                                      </el-button>
                                  </el-input>
                              </el-form-item>
                              <el-form-item class="L-form-item" prop="phone">
                                  <el-input :disabled="!phone" v-model="addForm.userPhone" class="L-input">
                                      <el-button slot="append" class="L-input-button" @click.stop.native="changeuserPhone">
                                          <img :src="changePhoneImgUrl" :alt="statesPhone" :title="statesPhone">
                                      </el-button>
                                  </el-input>
                              </el-form-item>
                              <el-form-item class="L-form-item" prop="userId">
                                  <el-input :disabled="true" v-model="addForm.userId"></el-input>
                              </el-form-item>
                              <el-form-item class="L-form-item" prop="roleName">
                                  <el-input :disabled="true" v-model="addForm.roleName"></el-input>
                              </el-form-item>
                              <el-form-item class="L-form-item" prop="userPass">
                                  <el-input :disabled="true" v-model="addForm.password" class="L-input" type="password">
                                      <el-button slot="append"  class="L-input-button" @click.stop.native="changePassword">
                                          <img src="../imgs/edit.png" alt="编辑">
                                      </el-button>
                                  </el-input>
                               </el-form-item>
                        </el-form>
                    </div>
                </el-col>
            </el-row>
        </div>

        <!-- 修改密码页 -->
        <el-dialog title="密码修改" v-model="passwordFormVisible" :close-on-click-modal="false" align="center" class="L-header">
            <el-form :model="editForm" label-width="80px" :rules="userEditFormRules" class="L-header-form" ref="editForm">
                <el-form-item label="原始密码" prop="oldPassword">
                    <el-input v-model="editForm.oldPassword" auto-complete="off" type="password"></el-input>
                </el-form-item>
                <el-form-item label="新  密  码" prop="newPassword">
                    <el-input v-model="editForm.newPassword" :span="24" type="password"></el-input>
                </el-form-item>
                <el-form-item label="确认密码" prop="confirmNewPassword">
                    <el-input v-model="editForm.confirmNewPassword" :span="24" type="password"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer" >
                <el-button class="L-edit-button L-save-password-button" type="primary" @click.native="saveSubmit" :loading="addLoading">保存</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
import { api } from '../js/api/api';
import { getCookie } from "../js/api/cookie";

export default {
    data() {
        let ifTheSame = (rule, value, callback) => {
            if(value != this.editForm.newPassword){
                return callback(new Error('两次密码输入不一致，重新输入'));
            }else{
                callback();
            }
        }
        let passLength = (rule, value, callback) => {
            if(value.length < 6 || value.length > 10){
                return callback(new Error('密码长度6-10位'));
            }else{
                callback();
            }
        }
        return {
            addLoading:false,
            addForm:{
                userName:"",
                userPhone:"",
                userId:"",
                role:"",
                password:"",
                sid:""
            },
            cookie:getCookie("token"),
            dialogVisible: false,
            passwordFormVisible:false,
            editLoading:false,
            editForm:{
                oldPassword:'',
                newPassword:'',
                confirmNewPassword:''
            },
            type:'',
            name:false,
            phone:false,
            imageUrl:'',
            imgurl:api.UserPhoto,
            statesPhone:"编辑",
            statesName:"编辑",
            changeNameImgUrl:'./imgs/edit.png',
            changePhoneImgUrl:'./imgs/edit.png',
            //验证
            menu:["姓名","电话","账号","角色","密码"],
            userEditFormRules:{
                name : [
                    { required: true, message: '请输入姓名', trigger: 'blur' }
                ],
                phone : [
                    { required: true, message: '请输入手机号/座机号', trigger: 'blur' }
                ],
                oldPassword: [
                    { required: true, message: '请输入原始密码', trigger: 'blur' }
                ],
                newPassword: [
                    { required: true, message: '请输入新密码', trigger: 'blur' },
                    { validator: passLength, trigger: 'blur'}
                ],
                confirmNewPassword: [
                    { required: true, message: '请确认密码', trigger: 'blur' },
                    { validator: ifTheSame, trigger: 'blur'}
                ]
            },
            nameState : false,
            phoneState : false
        }
    },
    mounted(){
        this.getAdmin()
    },
    methods:{
        getAdmin: function(){
            this.$http.post( api.getUser).then((res) => {
                this.addForm = res.body.rtData.user;
                this.addForm.roleName = res.body.rtData.role.roleName;
                this.imageUrl = encodeURI(encodeURI(api.getAttach + "&attachPath=" + res.body.rtData.userPhotos[0].attachPath));
            })
        },
        changeName:function(){
            this.addForm.userName  ?  this.nameState = true : this.nameState = false;
            this.$root.eventHub.$emit('EVENT_CHANGE_NAME', this.addForm.userName)
            if(this.nameState){
                this.name = !this.name;
                this.changeNameImgUrl = "./imgs/save.png"
                this.statesName = "保存"
                if(!this.name){
                    this.changeNameImgUrl = "./imgs/edit.png"
                    this.statesName = "编辑"
                    let para = {
                        sid : this.addForm.sid,
                        userName : this.addForm.userName
                    }
                    this.$http.post( api.changeUser, para).then((res) => {
                        this.getAdmin()
                    })
                }
            }
        },
        changeuserPhone:function(){
            this.addForm.userPhone  ?  this.phoneState = true : this.phoneState = false
            if(this.phoneState){
                this.phone = !this.phone;
                this.changePhoneImgUrl = "./imgs/save.png"
                this.statesPhone = "保存"
                if(!this.phone){
                    this.changePhoneImgUrl = "./imgs/edit.png"
                    this.statesPhone = "编辑"
                    let para = {
                        sid : this.addForm.sid,
                        userPhone : this.addForm.userPhone
                    }
                    this.$http.post( api.changeUser, para ).then((res) => {
                        this.getAdmin()
                    })
                }
            }
        },
        changePassword:function(){
            this.passwordFormVisible = true;
            this.editForm = {
                oldPassword:'',
                newPassword:'',
                confirmNewPassword:''
            }
        },
        saveSubmit:function(){
            this.$refs.editForm.validate((valid) => {
				if(valid){
					this.$confirm('确认提交吗？', '提示', {})
					.then(() => {
						let para = {
							sid: this.addForm.sid,
							oldPassword: this.editForm.oldPassword,
							password: this.editForm.newPassword,
						}
						this.$http.post( api.changeUser , para)
						.then((res) => {
                            console.log(res)
							this.$message({
								message: res.body.rtMsg
							})
							this.$refs['editForm'].resetFields();
							this.passwordFormVisible = false;
							// this.getAdmin();
						})
					})
				}
			});
        },
        handleAvatarScucess(res, file) {
            this.imageUrl = URL.createObjectURL(file.raw);
            this.$root.eventHub.$emit('EVENT_CHANGE_PHOTO',this.imageUrl)
        },
        beforeAvatarUpload(file) {
            const isJPG = file.type === ['image/jpeg','image/png'];
            // const isLt2M = file.size / 1024 / 1024 < 2;
            //
            // if (!isJPG) {
            //     this.$message.error('上传头像图片只能是 JPG 格式!');
            // }
            // if (!isLt2M) {
            //     this.$message.error('上传头像图片大小不能超过 2MB!');
            // }
            // return isJPG && isLt2M;
        }
    }
}
</script>

<style lang="css">
    .L-grid-content {
        border-radius: 4px;
        min-height: 36px;
    }
    .L-width{
        width: 80%;
        min-width: 1000px;
        margin:10% auto 0;
    }
    .L-background{
        width: 100%;
        height: 100%;
        background: #F2F2F2;
        padding: 1px;
    }
    .L-width .el-row{
        margin:0 auto;
    }
    .L-grid-content img{
        display: block;
        margin: auto;
    }
    .L-form-item{
        margin:0 !important;
    }
    .L-form-item input{
        text-align: center;
    }
    .L-form-item .el-form-item__label{
        width: 100%;
        height: 3.5rem;
        border-bottom: 1px solid #E4E4E4;
        text-align: center;
    }
    .L-input{
        position: relative;
    }
    .L-form-item input{
        background: #fff !important;
        height: 3.5rem;
    }
    .L-input .el-input-group__append{
        position: absolute;
        right: 1px;
        height: 2rem;
        top: 45%;
        width: 2rem;
        border: none;
        /*transform: translate(-50%,-50%);*/
        padding: 0 !important;
        background: transparent;
    }
    .L-input-button{
        padding: 0 !important;
    }
    .L-input-button img{
        width: 2rem;
        height: 2rem;
    }
    .L-header .el-dialog__header{
        background: #4DA1E7;
        height: 3rem;
    }
    .L-header .el-dialog__title{
        color:#fff;
    }
    .L-header .dialog-footer{
        width: 100%;
    }
    .L-header .L-save-password-button{
        width: 20%;
        height:4rem;
        margin: auto;
        display: block !important;
    }
    .L-header .el-dialog_body{
        padding: 3rem 0 !important;
    }
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
       border-color: #20a0ff;
    }
    .L-bg-purple .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 16rem;
        height: 16rem;
        line-height: 178px;
        text-align: center;
    }
    .L-bg-purple .avatar {
        width: 190px;
        height: 190px;
        display: block;
    }
    .L-header-form{
        position: relative;
    }
    .L-header-form .el-form-item__label{
        width: 7rem !important;
    }
    .L-header-form .el-form-item__label::before{
        position: absolute;
        left:0;
    }
    .L-header-form .el-form-item__content{
        width: 80%;
        margin-left: 7rem !important;
    }
</style>
