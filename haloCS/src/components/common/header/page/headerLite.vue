<template>
  <div class="header clearfix">
    <a class="logo" href="./index.html" target="_blank"><img src="http://img.tozlam.cn/halo-230-40-blue.png" width="120px" height="24px"></a>

    <div class="breadcrumb">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :class="{step:currStep=='购物车'}" >购物车</el-breadcrumb-item>
        <el-breadcrumb-item :class="{step:currStep=='确认订单'}">确认订单</el-breadcrumb-item>
        <el-breadcrumb-item :class="{step:currStep=='在线支付'}">在线支付</el-breadcrumb-item>
        <el-breadcrumb-item :class="{step:currStep=='完成'}">完成</el-breadcrumb-item>
      </el-breadcrumb>
    </div>

    <div class="login clearfix">
      <ul>
        <li v-if="isLogin">
          <el-dropdown trigger="hover" @command=" handleCommand" placement="bottom">
          <span class="el-dropdown-link">
          {{user}}
          </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="loginout" @click="goRouter(login)">我的订单</el-dropdown-item>
              <el-dropdown-item command="loginout" divided>地址管理</el-dropdown-item>
              <el-dropdown-item command="loginout" divided>退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </li>
        <li v-else>
          <span @click="goOther('login')">
          登录
          </span>
        </li>
        <li v-if="!isLogin">
          <span @click="goOther('register')">
            注册
          </span>
        </li>
      </ul>


    </div>
  </div>
</template>
<script>
  export default {
    props: {
      currStep: {
        type: [String,Number]
      },
    },
    data() {
      return {
        isLogin: false,
        user: "",
      }
    },
    methods: {
      goOther(that){
        sessionStorage.setItem('pageHistory',this.$route.fullPath);
        this.$router.push({path: "/" + that});
      },
      goRouter(that) {
        this.$router.push({path: "/" + that});
      },
      handleCommand(command) {
        if (command == 'loginout') {
          var url = this.$rootUrl + "/api/user/logout";
          const options = {
            method: 'GET',
            url: url,
            data: {}
          };
          this.$axios(options).then((res) => {
            let item = res.data.data;
            if (item.errorCode == 0) {
              sessionStorage.removeItem('expireTime');
              sessionStorage.removeItem('token');
              sessionStorage.removeItem('userName');
              sessionStorage.removeItem('avatar');
              const path = this.$route.path;
              if(path == '/newPost' || path == '/user' || path == '/mallCheck' || this.$route.matched[0].path == '/member'){
                this.$router.push({path:'/'});
              }else{
                this.isLogin = false;
              }
            }
          });
        }
      },
    },
    created(){
      const expireTime = sessionStorage.getItem('expireTime');
      const nowTime = new Date().getTime();
      if(expireTime && nowTime < expireTime){
        this.isLogin = true;
      }
      this.user = this.$store.state.userInfo.username;
    }
  }
</script>
<style lang="less">
  .header {
    z-index: 100;
    background-color: #fff;
  }

  .navs {
    float: right;
    margin-right: 150px;
    margin-top: 20px;
  }

  .header a {
    color: #000;
  }

  .header a:hover {
    color: #31a5e7;
  }

  .login {
    position: absolute;
    right: 150px;
    top: 20px;
    font-size: 14px;
  }

  .login a {
    padding-right: 5px;
    padding-left: 5px;
  }

  .navs li {
    width: 60px;
    display: inline-block;
    margin-right: 20px;
    font-size: 14px;
    cursor: pointer;
  }

  .nav-img {
    width: 100px;
  }

  .logo {
    position: absolute;
    margin-left: 175px;
    z-index: 100;
  }

  .logo img {
    margin-top: 20px;
    padding-bottom: 10px;

  }

  .header {
    position: relative;
    width: 100%;
    height: 60px;
    top: 0;
    border-bottom: 0.5px solid #f3f3f3;
  }

  .header:hover {
    background-color: #fff;
  }

  .nav-product dl {
    width: 100%;
    position: absolute;
    left: 0;
    padding-left: 300px;
    padding-top: 10px;
    margin-top: 0px;
    background-color: #fff;
    display: none;
  }

  .nav-product:hover dl {
    display: block;
  }

  .nav-product dl dd {
    float: left;
    width: 150px;
    height: 130px;
    text-align: center;

  }

  .nav-product dl dd p {
    color: #515151;
    position: relative;
    font-size: 12px;
  }

  a:hover {
    text-decoration: none !important;
  }

  .header {
    position: absolute;
    width: 100%;
    top: 0;
  }

  .nav-search {
    width: 135px !important;
    height: 25px;
    border-radius: 20px;
    border: 0.5px solid #dcdcdc;
  }

  .nav-search input {
    width: 100px;
    border: none;
    margin-left: 8px;
    margin-top: 6px;
    outline: none;
    font-size: 11px;
    background: transparent;
  }

  .nav-search i {
    margin-right: 3px;
  }

  .userIcon {
    border-radius: 100%;
  }

  .login li {
    float: left;
    width: 50px;
    cursor: pointer;
  }

  .breadcrumb {
    width: 600px;
    position: relative;
    top: 25px;
    left: 350px;
  }

  .step {

    span{
      color: #31a5e7;
    }
  }
</style>
