<template>
    <div>
        <!-- <div>{{$route.params.id}}</div> -->
        <div class="ypaifa" style="margin-bottom:10px;">
            <div class="yTimeAbian">
                <div class="ypaifaM">{{keeper.createTime}}</div>
                <div class="ypaifaM">订单编号：{{keeper.orderNumber}}</div>
            </div>
            <div>
                <el-button type="primary" @click="paifa">派发</el-button>
            </div>
        </div>
        <div class="ydingdanmain">
          <div style="border-bottom:1px solid #d1d1d1;">
              <div><b>收获地址：</b><span>{{keeper.receiveUserAddress}}</span></div>
              <div class="yAddressee">
                  <div><b>收件人：</b><span>{{keeper.receiveUserName}}</span></div>
                  <div><b>联系电话：</b><span>{{keeper.receiveUserPhone}}</span></div>
              </div>
          </div>
          <div  style="border-bottom:1px solid #d1d1d1;" class="yAddressee">
              <div>
                  <b>仓库管理员：</b>
                  <!-- <span>{{keeper.libKeeperName}}</span> -->
                  <el-select v-model="value" placeholder="请选择" clearable >
                      <el-option v-for="(item,key) in allCpeople" :label="item.userName" :value="item.sid" :key="item.userName"></el-option>
                  </el-select>
              </div>
              <div>
                  <b>联系电话：</b>
                  <!-- <span>{{keeper.libKeeperPhone}}</span> -->
                  <el-select v-model="selected" placeholder="请选择" clearable disabled >
                  </el-select>
              </div>
          </div>
          <div  style="border-bottom:1px solid #d1d1d1;margin-bottom:10px;" class="yAddressee">
              <div>
                  <b>施工人员：</b>
                  <span v-for="(item,key) in keeper.implementUserList">{{item.userName}}&nbsp;</span>
              </div>
              <div>
                  <b>联系电话：</b>
                   <span v-for="(item,key) in keeper.implementUserList">{{item.userPhone}}&nbsp;</span>
              </div>
          </div>
          <div style="border:1px solid #d1d1d1;">
              <div class="yChanpin" v-for="(item,key) in keeper.orderItemList">
                  <div style="border-right:1px solid #d1d1d1;border-bottom:1px solid #d1d1d1;" class="ychanPinN">
                        <div class="ychanPinNAi">
                                <img :src="item.productPhotos[0].attachPath" alt="" style="width:3rem;height:3rem;display:block;">
                            <div>
                                <span>{{item.productName}}</span>
                            </div>
                        </div>
                        <div>{{item.productNums}}</div>
                  </div>
                  <div style="border-bottom:1px solid #d1d1d1;" >
                        <span v-for="(equipmentId,key) in item.equipmentInfoList" >{{equipmentId.equipmentId}}/</span>
                  </div>
              </div>
          </div>
        </div>
    </div>
</template>
<script>
    import {api} from '../js/api/api.js';
    import {getCheckdate,trasSecond} from '../js/api/getNowDate.js';
    export default ({
        data () {
            return {
                value:"",
                value1:"",
                keeper:[],
                allCpeople:[]
            }
        },
        computed:{
            selected:{
                get () {
                    let that=this;
                    for(var i=0;i<that.allCpeople.length;i++){
                        if(that.value==that.allCpeople[i].sid){
                            return that.allCpeople[i].userPhone
                        }
                    }
                }
            }
        },
        mounted () {
            this.getInfo();
        },
        methods:{
            getInfo () {
                let id=this.$route.params.id
                this.$http.post(api.getCustomerOrderById,{"sid":id}).then((response)=>{
                    // console.log(response.body)
                    this.keeper=response.body.rtData
                    // this.keeper.createTime=trasSecond(this.keeper.createTime);
                    for(var i=0;i<this.keeper.orderItemList.length;i++){
                        this.keeper.orderItemList[i].productPhotos[0].attachPath=encodeURI(encodeURI(api.getAttach+"&attachPath="+this.keeper.orderItemList[i].productPhotos[0].attachPath+""));
                    }
                    this.value=this.keeper.libKeeperName

                })
                // 仓库人员
                this.$http.post(api.allSSpeople,{"roleId":4}).then((response)=>{
                    // console.log(response.body);
                    this.allCpeople=response.body.rows
                })
            },
            paifa () {
                if(!this.value){
                    this.$alert("仓库管理员不能为空");
                    return;
                }else{
                    let params={
                        sid:this.$route.params.id,
                        libKeeperId:this.value,
                        orderStatus:"待发货"
                    }
                    this.$http.post(api.updateCustomerOrder,params).then((response)=>{
                        // console.log(response.body)
                        if(response.body.rtState){
                            this.$message({
                                message:'派发成功',
                                type:"success"
                            });
                        }
                        this.$router.push("/OrderDistribution")
                    })
                }

            }
        }
    })
</script>
<style>
    .ychanPinNAi{
        padding-left: 10px;
        display: -webkit-box;
        display: -ms-flexbox;
        display: -moz-box;
        -ms-flex-align:center;
        -moz-box-align:center;
        -webkit-box-align:center;
        text-align: center;
    }
    .ychanPinNAi>div:last-child{
        -webkit-box-flex:1;
        -moz-box-flex:1;
        -ms-flex:1;
        width:0%;
    }
</style>
