<template>
  <div class="col-sm-12 col-md-6 col-lg-4 mx-auto">
    <div class="login-wrapper">
      <div class="login-header" style="margin-top: 75px;">
        <img src="../../../assets/img/logo.png" v-on:click="redirect('/')">
      </div>
      <span style="width:100%;float:left;text-align:center;font-size:20px;margin-bottom:20px;">
        Login to <b class="text-primary">{{config.APP_NAME}}</b>
      </span>
      <div class="login-message-holder login-spacer" v-if="errorMessage != null">
        <span class="text-danger"><b>Oops!</b> {{errorMessage}}</span>
      </div>
      <div>
        <div class="input-group login-spacer">
          <span class="input-group-addon" id="addon-1"><i class="fa fa-user"></i></span>
          <input type="text" class="form-control form-control-login" placeholder="Username or Email" aria-describedby="addon-1" v-model="username">
        </div>
        <div class="input-group login-spacer">
          <span class="input-group-addon" id="addon-2"><i class="fa fa-key"></i></span>
          <input type="password" class="form-control form-control-login" placeholder="********" aria-describedby="addon-2" v-model="password" @keyup.enter="logIn()">
        </div>
        <button class="btn btn-primary btn-block login-spacer" v-on:click="logIn()">Login</button>
        <button class="btn btn-warning btn-block login-spacer" v-on:click="redirect('/request_reset_password')">Forgot your Password?</button>
        <br>
        <div class="container-fluid separator">
            or
        </div>
        <br>
        <button class="btn btn-secondary btn-block login-spacer" v-on:click="redirect('/signup')">Create Account Now!</button>
      </div>
    </div>
    <div class="modal fade" id="otpModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-sm" role="document">
        <div class="modal-content">
          <div class="modal-header bg-primary">
            <h5 class="modal-title" id="exampleModalLabel">OTP Confirmation</h5>
            <button type="button" class="close" @click="cancelOTP()" aria-label="Close">
              <span aria-hidden="true" class="text-white">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <span v-if="otpErrorCode !== null" class="text-danger text-center">
                <label><b>Opps! </b>{{otpErrorCode}}</label>
            </span>
            <div class="form-group">
              <label for="exampleInputEmail1">OTP Code</label>
              <input type="text" class="form-control" placeholder="Type code here..." v-model="otpCode">
            </div>
          </div>
          <div class="modal-footer">
              <button type="button" class="btn btn-danger" @click="cancelOTP()">Cancel</button>
              <button type="button" class="btn btn-primary" @click="continueLoginViaOTP()">Continue</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.login-wrapper{
  width: 80%;
  margin: 0 10% 50px 10%;
}

.login-header{
  height: 100px;
  color: #006600;
  width: 100%;
  float: left;
  text-align: center;
}

.login-header img{
  height: 100px !important;
  width: 100px !important;
}

.login-header img:hover{
  cursor: pointer;
}

.login-message-holder{
  min-height: 30px;
  font-size: 12px;
  float: left;
  overflow: hidden;
}

.login-spacer{
  margin-bottom: 10px;
}/*-- login-spacer --*/

.forgot-password a{
  color: #006600 !important;
}
.forgot-password a:hover{
  cursor: pointer !important;
  text-decoration: underline !important;
  color: #009900 !important;
}

.btn{
  height: 50px !important;
}

.input-group-addon{
  width: 50px;
}

.form-control-login{
  height: 45px !important;
}

/*    Line with text on top  */
.separator>*{
  display: inline-block;
  vertical-align: middle;
}

.separator {
    text-align: center;
    border: 0;
    white-space: nowrap;
    display: block;
    overflow: hidden;
    padding: 0;
    margin: 0;
}

.separator:before, .separator:after {
    content: "";
    height: 1px;
    width: 50%;
    background-color: #ccc;
    margin: 0 5px 0 5px;
    display: inline-block;
    vertical-align: middle;
}

.separator:before {
    margin-left: -100%;
}

.separator:after {
    margin-right: -100%;
}

@media (max-width: 992px){
  .login-wrapper{
    width: 96%;
    margin: 0 2% 0 2%;
  }
}
</style>
<script>
import ROUTER from '../../../router'
import AUTH from '../../../services/auth'
import CONFIG from '../../../config.js'
export default {
  mounted(){
  },
  data(){
    return {
      username: null,
      password: null,
      errorMessage: null,
      user: AUTH.user,
      tokenData: AUTH.tokenData,
      otpCode: null,
      otpErrorCode: null,
      config: CONFIG
    }
  },
  methods: {
    logIn(){
      if(this.username !== null && this.username !== '' && this.password !== null && this.password !== ''){
        $('#loading').css({'display': 'block'})
        AUTH.authenticate(this.username, this.password, (response) => {
          this.errorMessage = null
          $('#loading').css({'display': 'none'})
        }, (response, status) => {
          $('#loading').css({'display': 'none'})
          this.errorMessage = (status === 401) ? 'Username and Password did not matched.' : 'Cannot log in? Contact us through email: ' + this.config.APP_EMAIL
        })
      }else{
        this.errorMessage = 'Please fill up all the required fields.'
      }
    },
    redirect(parameter){
      ROUTER.push(parameter)
    },
    request(parameter){
      this.APIRequest(parameter, {}).then(response => {
      })
    },
    cancelOTP(){
      AUTH.deaunthenticate()
      $('#otpModal').modal('hide')
    },
    continueLoginViaOTP(){
      // compare otp here
      let parameter = {
        condition: [{
          value: this.otpCode,
          column: 'code',
          clause: '='
        }, {
          value: AUTH.otpDataHolder.userInfo.id,
          column: 'account_id',
          clause: '='
        }]
      }
      $('#loading').css({'display': 'block'})
      this.APIRequest('notification_settings/retrieve', parameter).then(response => {
        $('#loading').css({'display': 'none'})
        if(response.data.length > 0){
          this.otpErrorCode = null
          $('#otpModal').modal('hide')
          AUTH.proceedToLogin()
        }else{
          // display invalid code here.
          this.otpErrorCode = 'Invalid OTP Code.'
        }
      })
    }
  }
}
</script>
