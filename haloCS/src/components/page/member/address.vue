<template>
  <div>
    <div class="addressWrite">
      <h3 class="write_title">{{content}}</h3>
      <v-write :form="form" :type="type" :length="addrDate.length" @ok="updateDate"></v-write>
    </div>
    <div class="addressHave">
      <h3 class="addressWrite">已有地址</h3>
      <div class="order_table_header">
        <ul class="clearfix">
          <li style="width: 150px">收货人姓名</li>
          <li style="width: 450px">收货人地址</li>
          <li style="width: 200px">收货人手机号</li>
          <li style="width: 180px">操作</li>
        </ul>
      </div>
      <el-table
        :data="addrDate"
        class="addr_table">
        <el-table-column
          prop="name"
          width="150"></el-table-column>
        <el-table-column
          prop="address"
          width="450"></el-table-column>
        <el-table-column
          prop="phone"
          width="200"></el-table-column>
        <el-table-column
          width="180">
          <template slot-scope="scope">
            <span class="blue" @click="updateAddr(scope.$index, scope.row)">修改</span>
            <span style="color:#ccc;cursor: pointer" @click="delAddr(scope.$index, scope.row)">删除</span>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>
<script>
  import vWrite from "./common/addressWrite";

  export default {
    data() {
      return {
        content: "新增收货地址",
        form: {id:"",name: "", phone: "", address: ""},
        addrDate: [],
        updateSelected: -1,
        type:1
      }
    },
    components: {
      vWrite,
    },
    methods: {
      updateDate(msg){
        if(msg == 1){
          this.form = {id:"",name: "", phone: "", address: ""};
          this.content = "新增收货地址";
          this.getData();
          this.$message({
            message: '收货人信息更新成功',
            type: 'success'
          });
          this.type = 1;
        }
      },
      delAddr(index, row){
        var id=this.addrDate[index].id
        var url = this.$rootUrl + "/api/user/delAddress";
        const options = {
          method: 'POST',
          url: url,
          data: {
            id:id
          }
        };
        this.$axios(options).then((res) => {
          let item = res.data.data;
          if (item.data) {
            if (item.errorCode == 0) {
              this.getData();
            }else if (item.errorCode == 403) {
              sessionStorage.setItem('pageHistory', this.$route.fullPath);
              this.$router.push({path: "/login"});
              throw item.msg;
            } else {
              throw item.msg;
            }
          }
        }).catch(errorMsg => {
          this.$message.error(errorMsg);
        });
      },
      updateAddr(index, row) {
        this.form = this.addrDate[index]
        this.content="修改收货地址"
        this.type=2
      },
      getData(){
        var url = this.$rootUrl + "/api/user/getAddress";
        const options = {
          method: 'GET',
          url: url,
          data: {}
        };
        this.$axios(options).then((res) => {
          let item = res.data.data;
          if (item.data) {
            if (item.errorCode == 0) {
              this.addrDate = item.data.address;
            }else if (item.errorCode == 403) {
              sessionStorage.setItem('pageHistory', this.$route.fullPath);
              this.$router.push({path: "/login"});
              throw item.msg;
            } else {
              throw item.msg;
            }
          }
        }).catch(errorMsg => {
          this.$message.error(errorMsg);
        });
      }
    },
    updated() {
      window.scroll(0, 0);
    },
    created(){
      this.getData()
    },
    watch: {
      '$route'(to, from) {
        this.getData()
      }
    }
  }
</script>
<style scoped>
  .addressWrite {
    margin-top: 20px;
    color: #666666;
  }

  .addressHave {
    position: relative;
  }

  .order_table_header {
    width: 100%;
    height: 40px;
    margin-top: 20px;
    background-color: #fafafc;
    display: flex;
    align-items: center;
    z-index: 100;
    position: absolute;
  }

  .order_table_header li {
    float: left;
    text-align: center;
    font-size: 14px;
    color: #666666;
  }

  .addr_table {
    position: absolute;
    top: 60px;
    width: 100%;
    text-align: center;
  }

  .blue {
    cursor: pointer;
    display: inline-block;
  }

  .blue:first-child {
    margin-right: 10px;
  }

  .blue:last-child {
    margin-left: 10px;
  }

  .blue:hover {
    color: #00a7ea !important;
  }
</style>
