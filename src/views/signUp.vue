<template>
  <div class="box">
    <div class="sign-form">
      <van-field v-model="phoneNum" placeholder="请输入您的手机号码" :left-icon="icon.firIcon" />
      <div class="opt-box">
        <van-field v-model="optNum" placeholder="请输入图形验证码" :left-icon="icon.twoIcon"/>
        <img :src="imgUrl" alt @click="getCode">
      </div>
      <div class="opt-box">
        <van-field v-model="emailOptNum" placeholder="请输入短信验证码" :left-icon="icon.thirdIcon"></van-field>
        <van-button
          class="send-opt"
          :class="{disabled: !this.canClick}"
          :disabled="!this.canClick"
          slot="button"
          size="small"
          @click.native="sendOpt"
        >{{content}}</van-button>
      </div>
      <van-field v-model="passwordNum" type="password" placeholder="请设置登录密码(6-16位字母和数字)" :left-icon="icon.fourIcon"/>
      <!-- <span class="sp">{{passwordValidate.errorText}}</span> -->
      <van-field v-model="passwordCheckNum" type="password" placeholder="请确认登录密码(6-16位字母和数字)" :left-icon="icon.fourIcon"/>
      <!-- <span class="sp">{{passwordCheckValidate.errorText}}</span> -->
      <van-button class="sign-btn" @click="signNow" :loading="isLoad" type="info">立即注册</van-button>
      <van-checkbox v-model="checked"  class="protocol">已阅读并同意 <span class="protocol-sp"><a href="http://login.ceyu99.com/guide/index.html">《服务协议》</a></span></van-checkbox>
    </div>

    <div class="download-footer">
      <div class="img_box">
        <img src="../assets/login_image_logo3x.png" alt>
      </div>
      <div class="text-hx">
        <p>期宝通</p>
        <p>闪电开户·极速下单</p>
      </div>
      <div class="download-btn" @click="goDownLoad">下载APP</div>
    </div>
  </div>
</template>

<script>
import { setTimeout } from "timers";
import util from "../util/util";
import api from "../util/api";
import { encode } from "punycode";
import axios from "axios";
import sha1 from "js-sha1";
let tenantId = util.getParam("TENANT_ID")
// console.log(tenantId)
let inviteCode = util.getParam("inviteCode")

export default {
  components: {},
  props: {},
  data() {
    return {
      icon: {
        firIcon: require("../assets/login_icon_phone2x.png"),
        delIcon: require("../assets/close@2x.png"),
        twoIcon: require("../assets/login_icon_invite2x.png"),
        thirdIcon: require("../assets/login_icon_verification2x.png"),
        fourIcon: require("../assets/login_icon_password2x.png")
      },
      content: "发送验证码", // 按钮里显示的内容
      totalTime: 60, //记录具体倒计时时间
      canClick: true,
      phoneNum: "", //手机号码
      optNum: "", //图形验证码
      emailOptNum: "",//短信验证码
      passwordNum: "", //设置密码
      passwordCheckNum:"",//确认密码
      isLoad: false,
      imgUrl: "", //后台返回的图形验证码
      randomNum: "",
      passwordFlag:false,
      checked:false
    };
  },
  watch: {},
 computed:{
    //   passwordValidate: function() {
    //     var errorText;
    //     if(!/^[0-9A-Za-z]{6,15}$/.test(this.passwordNum)) {
    //         errorText = ''
    //     } else {
    //         errorText = ''
    //     }
    //     if(!this.passwordFlag) {
    //         errorText = ''
    //         this.passwordFlag = true
    //     }
    //     return {
    //         errorText
    //     }
    // },
    //     passwordCheckValidate: function() {
    //     var errorText;
    //     if(!/^[0-9A-Za-z]{6,15}$/.test(this.passwordCheckNum)) {
    //         errorText = ''
    //     }else if(this.passwordCheckNum !==this.passwordNum ){
    //         errorText = ''
    //     }else {
    //         errorText = ''
    //     }
        
    //     if(!this.passwordFlag) {
    //         errorText = '';
    //         this.passwordFlag = true
    //     }
    //     return {
    //         errorText
    //     }
    // }
 },
  methods: {
  
    getCode() {
      this.randomNum = util.randomLenNum(4, true);
      axios
        .get(`http://192.168.1.124:9999/code?randomStr=${this.randomNum}`, {
          responseType: "arraybuffer" //指定返回数据的格式为blob
        })
        .then((res) => {
          var src =
            "data:image/jpg;base64," +
            btoa(
              new Uint8Array(res.data).reduce(
                (data, byte) => data + String.fromCharCode(byte),
                ""
              )
            );
          this.imgUrl = src; //图片回显
          // let blob = new Blob([res.data], { type: 'image' });
          // this.imgUrl = (window.URL || window.webkitURL).createObjectURL(blob);
          // console.log(res, encode(res.data), 1111);
        })
        .catch();
    },
    sendOpt() {
      if (this.phoneNum.trim() == ""  ){
          this.$toast("手机号码不能为空");
        return;
        } else if( !/^[1][3,4,5,7,8][0-9]{9}$/.test(this.phoneNum)){
        this.$toast("请填写正确手机号码");
        return;
      } else if (this.optNum.trim() == ""|| this.optNum.length > 4) {
        this.$toast("请填写正确图形验证码");
        return;
      } 
      axios
        .get("/user/mobile/sendSmsCode",{params:{ 
          randomStr: this.randomNum,
          mobile: this.phoneNum,
          captcha: this.optNum,
          type: 1
        },  
        headers: {
             "TENANT_ID": tenantId,
        }
         })
        .then((res) => {
          // console.log(res)
          if(res.data.code=="10011047"){
            this.$toast("请填写正确图形验证码");
            this.getCode()
            return
          }else if(res.data.code=="10011013"){
            this.$toast("手机号已存在");
            return
          }else if(res.data.code=="10011044"){
            this.$toast("发送验证码频繁，请稍后再试")
          }
         this.$toast("发送手机验证码成功, 请注意查收");
         if (!this.canClick) return; //改动的是这两行代码
         this.canClick = false;
         this.content = this.totalTime + "s后重新发送";
         let clock = window.setInterval(() => {
         this.totalTime--;
         this.content = this.totalTime + "s后重新发送";
         if (this.totalTime < 0) {
          window.clearInterval(clock);
          this.content = "重新发送验证码";
          this.totalTime = 60;
          this.canClick = true; //这里重新开启
        }
      }, 1000);
          
        })
        .catch();
    },
    signNow() {
      if (
        this.phoneNum.trim() == "" ||
        !/^[1][3,4,5,7,8][0-9]{9}$/.test(this.phoneNum)
      ) {
        this.$toast("请按规则填写手机号码");
        return;
       }else if (this.optNum.trim() == "" || this.optNum.length > 4) {
        this.$toast("请填写正确图形验证码");
        this.getCode()
        return;
      } else if (this.emailOptNum.trim() == "") {
        this.$toast("请填写短信验证码");
        return;
      } else if (this.passwordNum.trim() == ""||!/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,15}$/.test(this.passwordNum)) {
        this.$toast("输入的密码格式不正确");
        return;
      }else if(this.passwordCheckNum.trim() == ""||!/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,15}$/.test(this.passwordCheckNum)){
        this.$toast("确认密码有误");
        return;
      }else if(this.passwordCheckNum !==this.passwordNum){
        this.$toast("两次密码不一致");
        return;
      }else if(this.checked== false){
        this.$toast("请阅读并同意服务协议");
        return;
      }
      this.isLoad = true;
      axios
        .post("/user/user/share/register", 
        {
          // TENANT_ID: this.$store.state.user.tenantId,
          // inviteCode: this.$store.state.user.inviteCode,
          // tenantId: tenantId,
          inviteCode: inviteCode,
          mobile: this.phoneNum,
          smsCode: this.emailOptNum,
          password: sha1(this.passwordNum),
          passwordCheck:sha1(this.passwordCheckNum),

        },
        {
        headers: {
             "TENANT_ID": tenantId,
        }
    },
)
        .then((res) => {
          if(res.data.code=="10011045"){
            this.$toast("手机验证码错误");
            return
          }
          this.isLoad = false;
          this.phoneNum="";
          this.optNum="";
          this.emailOptNum="";
          this.passwordNum="";
          this.passwordCheckNum="";
          this.$toast("注册成功");
          setTimeout(() => {
            var u = navigator.userAgentn;
            var isAndroid =
              u.indexOf("Android") > -1 || u.indexOf("Linux") > -1; //g
            var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
            if (isAndroid) {
              //这个是安卓操作系统
              window.location.href =
                "http://47.99.187.28:8083/qbt_loc-V1.0-2019-04-25-debug.apk";
            }
            if (isIOS) {
              //这个是ios操作系统
              window.location.href = "https://fir.im/QiBaoTong";
            }
          }, 2000);
        })
        .catch(() => {
          this.isLoad = false;
        });
    },
    goDownLoad() {
      var u = navigator.userAgent;
      var isAndroid = u.indexOf("Android") > -1 || u.indexOf("Linux") > -1; //g
      var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
      if (isAndroid) {
        //这个是安卓操作系统
        window.location.href =
          "http://47.99.187.28:8083/qbt_loc-V1.0-2019-04-25-debug.apk";
      }
      if (isIOS) {
        //这个是ios操作系统
        window.location.href = "https://fir.im/QiBaoTong";
      }
    }
  },
  created() {
    this.getCode();
  },
  mounted() {}
};
</script>



<style lang="scss" scoped>
.box {
  // height: 100%;
  min-height:100vh;
  background: #15181f;
  .sign-form {
    // overflow: hidden;
    padding: 0 0.53rem;
    margin-bottom: 2.3rem;
    // height: 100%;
    height:auto;
    .van-cell:not(:last-child)::after {
      border: 0;
    }
    .protocol{
     font-family: PingFangSC-Regular;
      font-size: 15px;
     color: #656C80;
      letter-spacing: 0;
      text-align: center;
      padding-top: 0.25rem;
      .protocol-sp{
        a{ 
          font-size:15px;
          color: #1E8CE1;
          text-decoration:none;
           font-family: PingFangSC-Regular;
           letter-spacing: 0;
        }
      }
      .van-checkbox__icon .van-icon{
        width: 15px;
        height: 15px;
      }
    }
    .sp{
      font-size: 0.3rem;
    color: #969799;
    /* margin-top: 31px; */
    position: fixed;
    padding-left: 36px;
    padding-top: 3px;
    }
    .opt-box {
      display: flex;
      img {
        height: 44px;
        min-width: 3.0rem;
        cursor: pointer;
        margin-top: 50px;
      }
      .send-opt {
        min-width: 3.0rem;
        // padding: 0 0.5rem;
        overflow: hidden;
        height: 44px;
        background: #242a3f;
        line-height: 44px;
        color: #e5eaf9;
        border: 0;
        margin-top: 50px;
      }
      .disabled {
        // background-color: #ddd;
        border-color: #ddd;
        color: #57a3f3;
        cursor: not-allowed; // 鼠标变化
      }
    }
    .van-cell {
      margin: 50px 0 0;
      background: #191c26;
      border-radius: 8px;
      color: #e5eaf9 !important;
      input {
        color: #e5eaf9 !important;
      }
    }
    .sign-btn {
      width: 100%;
      margin-top: 1.2rem;
    }
  }
}

.download-footer {
  width: 100%;
  position: fixed;
  bottom: 0;
  height: 2.13rem;
  background: #0e121d;
  div {
    display: inline-block;
    vertical-align: top;
    img {
      margin: 0.506rem 0 0.506rem 0.53rem;
      width: 1.13rem;
      height: 1.13rem;
    }
  }
  .text-hx {
    margin-left: 0.51rem;
    p:nth-child(1) {
      margin: 0.49rem 0 0;
      color: #ffffff;
      font-size: 0.45rem;
    }
    p:nth-child(2) {
      margin: 0;
      color: #858eb1;
      font-size: 0.34rem;
    }
  }
  .download-btn {
    float: right;
    margin: 0.57rem 0.53rem 0 0;
    font-size: 0.4rem;
    color: #fff;
    width: 3.18rem;
    height: 1.06rem;
    line-height: 1.06rem;
    text-align: center;
    background: #1e8ce1;
  }
}
</style>
<style scoped>
/* .box{
  height: 100%;
  background: #15181f;
}
.sign-form{
  overflow: hidden;
  padding: 0 .53rem;
  height: 100%;
} */

.sign-form >>> .van-field__control {
  color: #e5eaf9 !important;
}
.sign-form >>> .van-field__left-icon {
  display: flex;
  align-items: center;
}
</style>
