<template>
    <div id="w-login" @keyup.13="submitForm('ruleForm')">
		<el-form  :model="ruleForm" :rules="rules" ref="ruleForm"  label-position="left" label-width="0px" class="demo-ruleForm login-container">
			<h3 class="title">登录</h3>
			<el-form-item class="user" style="margin-bottom:30px;" prop="userId">
				<img src="../imgs/w-user.png" class="w-user">
				<el-input type="text" v-model="ruleForm.userId" id="userId" name ='userId' auto-complete="off" placeholder="请输入账号" ></el-input>
			</el-form-item>
			<el-form-item style="margin-bottom:20px;" prop="password" class="user">
				<img src="../imgs/w-password.png" class="w-user">
				<el-input type="password"  v-model="ruleForm.password"  auto-complete="off" placeholder="请输入密码"></el-input>
			</el-form-item>
			<el-form-item style="margin:10px 0;">
				<el-checkbox label="记住密码" name="type" v-model="checked"></el-checkbox>
			</el-form-item>
			<el-form-item style="width:100%;margin-top:30px;">
				<el-button type="primary" style="width:100%;background:#28A9FF;"  @click="submitForm('ruleForm')">登录</el-button>
			</el-form-item>
			<h2 class="w-h2">天创DM运营管理平台</h2>
		</el-form>
    </div>
</template>
<script>
import Vue from 'vue';
import { api } from '../js/api/api';
import { setCookie ,getCookie } from "../js/api/cookie.js";
	export default{
		   data() {
			return {
				ruleForm: {
					userId: '',
					password:""
				},
				checked:false,
				rules: {
					userId: [
						{ required: true, message: '请输入账号', trigger: 'blur' }
					]
				},
				arr:''
			};
			},
			created:function(){
				var userId=getCookie("userId");
				var password = getCookie("password");
				if(password ==null || password==undefined){
					password="";
				}
				var checked = getCookie("checked");
				if(checked){
					this.ruleForm.userId=userId;
					this.ruleForm.password = password;
					this.checked = true;
				};
			},
			methods: {
				submitForm(formName) {
					this.$refs[formName].validate((valid) => {
					if (valid) {
						this.$http.post(api.DoLogin,this.ruleForm).then(function(data){
							var json = data.body;
							// console.log(json);
							if(json.rtState){
								setCookie("roleId",json.rtData.user.roleId,30);
								setCookie("userName",json.rtData.user.userName,30);
								setCookie('token',json.rtData.token,1);
								setCookie("checked",this.checked,30);
                                sessionStorage.clear("active");
                                sessionStorage.clear("activeT");
								  Vue.http.headers.common['Token'] = json.rtData.token;
								this.$message({
									message:json.rtMsg,
									type:'success'
								});
								if(this.checked){
									setCookie("userId",this.ruleForm.userId,30);
									setCookie("password",this.ruleForm.password,30);
								}else{
									setCookie("userId",this.ruleForm.userId,-1);
									setCookie("password",this.ruleForm.password,-1);
									setCookie("checked",this.checked,-1);
								};
                                this.getMenu();
							}else{
								this.$message({
									message:json.rtMsg,
									type:'error'
								});
							}
						});
					} else {
						this.$message({
							message:"请输入账号",
							type:'error'
						});
						return false;
					}
					});
				},
				getMenu:function(){
					// var TOKEN = getCookie("token");
					// console.log(TOKEN);
					// this.$router.push("/main")
					this.$http.post(api.allPath).then((res)=>{
						var json =res.body;
						// alert("1");
						if(json.rtState){
							// console.log(json)
							var array = json.rtData[0];
							// console.log(array);
							this.arr = array.childMenu[0].menuUrl;
							// console.log(this.arr)
							this.$router.push(this.arr);
						}
					})
				}
			}
		};
</script>
<style>
    #w-login{
		width:100%;
		height:100%;
		background:url(.././imgs/w-login.png);
		background-size:100% 100%;
		position:relative;

	}
	.login-container {
    /*box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);*/
		width:26rem;
		height:34rem;
		background:#fff;
		position:absolute;
		top:50%;
		right:10%;
		transform:translate(0,-57%);
		z-index:999;
		padding:0 1.4rem;
		border-radius: 5px;
	}
	.title{
		height:2.5rem;
		line-height:2.5rem;
		font-size:1.4rem;
		color:#28A9FF;
		width:100%;
		margin-top:1.5rem;
		border-bottom:2px solid #ccc;
	}
	.user{
		margin-top:2.58rem;
	}
	.user input{
		padding:3px 0 3px 2.5rem;
	}
	.w-user{
		width:1.5rem;
		height:1.5rem;
		position:absolute;
		left:8px;
		top:8px;
		z-index:9999;
	}
	.w-h2{
		border-top:2px solid #ccc;
		margin-top:2.5rem;
		text-align:center;
		height:5rem;
		line-height:5rem;
		font-size:1.75rem;
		color:#28A9FF;
		font-family:'AdobeHeitiStd-Regular';
		font-weight:bold;
	}
</style>
