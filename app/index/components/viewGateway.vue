<template lang="html">
    <section class="L-background" style="overflow-y:auto;">
        <div class="L-center-div">
            <!-- 表格 -->
            <div class="L-header-div">
                <!-- <el-button type="primary" class='L-right-button' @click.native="">新增</el-button> -->
                <el-col :span="24" class="toolbar" style="margin: 10px 0;">
                    <el-form :inline="true">
                        <p> {{ this.$route.params.gatewayName }} </p>
                        <el-form-item style="width:11%;">
                            <el-button type="primary" class="L-button-content L-bindmessage-button" :disabled="this.sels.length === 0" @click.native="moveGateway">迁移</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
            </div>

            <!-- tabel表单 -->
            <div class="L-content-div">
                <el-table  :data="tableData"  border style="width: 100%" @selection-change="selsChange">
                    <el-table-column type="selection" width="55">
                    </el-table-column>
                    <el-table-column prop="key" label="序号" width="80">
                    </el-table-column>
                    <el-table-column  prop="equipmentName" label="设备名称" style="width:10%;">
                    </el-table-column>
                    <el-table-column prop="equipmentId" label="设备ID" style="width:10%;">
                    </el-table-column>
                </el-table>
            </div>
            <!-- 分页 -->
            <el-pagination class="L-pagination" layout="total, prev, pager, next" @current-change="currentChange"  :current-page="this.page.currentPage" :total="this.page.total" :page-size="this.page.rows">
    		</el-pagination>
        </div>
        <!-- 迁移页面 -->
        <el-dialog  title="迁移" v-model="moveVisible" :close-on-click-model="false" class="L-header">
            <label style="font-size:1.5rem;margin-right:0.8rem;">网关名称</label>
            <el-select v-model="value" placeholder="请选择" class="L-select">
                <el-option v-for="item in options" :label="item.gatewayName" :value="item.sid" :key="item.gatewayName">
                </el-option>
            </el-select>
            <div slot="footer" class="dialog-footer" >
                <el-button class="L-edit-button L-save-password-button" type="primary" @click.native="saveGateway" :loading="editLoading">保存</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
import { api } from '../js/api/api';
export default {
    data(){
        return{
            addLoading:false,
            moveVisible:false,
            editLoading:false,
            page:{
                page:1,
                total:0,
                rows:15,
                sort:"sid",
                order:"desc",
                gatewaySid:this.$route.params.id
            },
            gateway:false,
            tableData:[],
            sels:[],
            options:[],
            value:""
        }
    },
    mounted(){
        this.moveGatewayList()
        this.getAllGateway()
    },
    watch:{
        '$route':['gatewayName','id']
    },
    methods:{
        moveGatewayList: function(){
            let para = Object.assign({}, this.page)
            this.$http.post( api.queryEquipmentListByGatewaySid , para).then((res) => {
                this.tableData = res.body.rows;
                this.page.total = res.body.total
                for(var key in this.tableData){
                    this.tableData[key].key = ((this.page.page - 1) * 15) + (key * 1 + 1);
                }
            })
        },
        selsChange:function(sels){
            this.sels = sels;
        },
        getAllGateway:function(){
            this.$http.post( api.getAllGateway , {gatewaySid:this.$route.params.id}).then((res) => {
                this.options = res.body.rtData;
                this.page.total = res.body.total
            })
        },
        moveGateway:function(){
            this.moveVisible = true;
        },
        saveGateway:function(){
            var ids = this.sels.map(item => item.sid).toString();
            if(this.value){
                this.$confirm('确认迁移选中的网关吗？', '提示', {
    				type: 'warning'
    			}).then(() => {
    				this.listLoading = true;
    				let para = { gatewaySid: this.value , equipmentIds: ids };
    				this.$http.post( api.exchangeGateway , para).then((res) => {
    					this.listLoading = false;
    					this.$message({
    						title: '成功',
    						message: res.body.rtMsg,
    						type: 'success'
    					});
                        this.moveVisible = false;
    					this.moveGatewayList();
    				});
    			})
            }
        },
        currentChange:function(val){
            this.page.page = val;
            this.moveGatewayList();
        }
    }
}
</script>

<style lang="css">
    .L-header-div p{
        text-align: left;
        font-size: 20px;
        width: 10%;
        float: left;
        line-height: 2;
    }
    .el-icon-caret-top:before{
        color: #4DB3FF;
    }
    .L-select{
        width: 40%;
    }
    .L-select .el-input{
        width: 100%;
    }
</style>
