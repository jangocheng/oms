<template>
    <div style="height:100%;background:#f2f2f2;padding:20px;overflow-y:scroll;">
		<!--列表-->
      <el-table :data="systemLog" border center style="width: 100%" @selection-change="handleSelectionChange" @current-change="handleCurrentChange">
          <el-table-column prop="key" label="序号" width="80"  center></el-table-column>
          <el-table-column prop="operateUserName"  label="操作人" center></el-table-column>
          <el-table-column prop="operateTime" label="创建时间"  width="200" :formatter="formatTime"  align='center'>
		    </el-table-column>
          <el-table-column  prop="logType" label="操作类型"  show-overflow-tooltip center width="120"></el-table-column>
          <el-table-column  prop="ip" label="IP地址"  show-overflow-tooltip center></el-table-column>
          <el-table-column  prop="remark" label="描述"  show-overflow-tooltip center width="250"></el-table-column>
          <el-table-column  prop="operation" label="操作"  center>
              <template scope="scope">
                  <el-button type="text" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
              </template>
          </el-table-column>
        </el-table>
        <el-col :span="24" class="w-toolbar" style="height:5rem;">
              <el-pagination   style="text-align:right;"  :current-page="currentPage"  layout= " total ,prev, pager, next" :total="total"  @current-change="currentChange" :page-size="pageSize"></el-pagination>
        </el-col>
    </div>
</template>
<script>
  import { api } from '../js/api/api';
  export default {
    data() {
      return {
          systemLog: [],
          multipleSelection: [],
          currentPage:1,
          total:9,
          pageSize:15,
          index:234
        }
      },
      mounted () {
            this.handleList();
      },
      methods: {
        formatTime: function (row, column) {
          Date.prototype.format = function(format) {
            var date = {
              "M+": this.getMonth() + 1,
              "d+": this.getDate(),
              "h+": this.getHours(),
              "m+": this.getMinutes(),
              "s+": this.getSeconds(),
              "q+": Math.floor((this.getMonth() + 3) / 3),
              "S+": this.getMilliseconds()
            };
            if (/(y+)/i.test(format)) {
              format = format.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length));
            }
            for (var k in date) {
              if (new RegExp("(" + k + ")").test(format)) {
                  format = format.replace(RegExp.$1, RegExp.$1.length == 1 ? date[k] : ("00" + date[k]).substr(("" + date[k]).length));
              }
            }
            return format;
          };
          return new Date(row.operateTime).format('yyyy-MM-dd');
        },
        handleSelectionChange(val) {
          this.multipleSelection = val;
        },
        handleCurrentChange(val) {
        this.currentRow = val;
        this.handleList();
      },
      //删除列表数据
        handleDel:function(index,row){
          this.$confirm('确认删除该记录吗?', '提示', {
            type: 'warning'
          }).then(() => {
            let para = { sid: row.sid };
                this.$http.post(api.DelSystemLog,para).then(function(data){
                    var json = data.body;
                    // console.log(json);
                    if(json.rtState){
                        this.$message({
                          message:json.rtMsg,
                          type:"success"
                        })
                         this.handleList();
                      }else{
                        this.$message({
                          type:"error",
                          message:json.rtMsg
                        })
                      }
                })
          }).catch(() => {
            //点击取消按钮
          });
        },
        //渲染列表数据
        handleList:function(){
            let para={
                "rows":this.pageSize,
                "page":this.currentPage,
                "sort":"sid",
                "order":"desc"
            };
            this.$http.post(api.SystemLog,para).then(function(res){
                var json = res.body;
                this.systemLog = json.rows;
                for(var key in this.systemLog){
                    this.systemLog[key].key = ((this.currentPage - 1) * 15) + (key * 1 + 1);
                }
                this.total=json.total;
            })
        },
        currentChange (val) {
            this.currentPage=val;
            this.handleList();
        }
      }
  }
</script>
<style>
      .w-toolbar .el-pagination{
        padding:0;
    }
</style>
