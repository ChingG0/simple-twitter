<template>
  <div class="container">
    <div class="header">
      <img src="./../assets/logo.png" alt="logo" class="logo">
      <h3>後台登入</h3>
    </div>
    <form class="" @submit.prevent.stop="handleSubmit">
      <div class="form-label-group" :class="{'errorAccount': errorAccount}">
        <label for="account">帳號</label>
        <input id="account" name="account" autofocus v-model="account" type="name" placeholder="請輸入帳號"
          autocomplete="account">
      </div>
      <div class="form-label-group" :class="{'errorPassword': errorPassword}">
        <label for="password">密碼</label>
        <input id="password" name="password" v-model="password" type="password" placeholder="請輸入密碼"
          autocomplete="password">
      </div>
      <button class="btn">登入</button>
    </form>
    <div class="other-link d-flex justify-content-end">
      <router-link to='admin/signin'>
      <div class="btn-cancel"><u>前台登入</u></div>
    </router-link>
    </div>
  </div>
</template>

<script>
import adminAPI from '../apis/admin'
import { Toast } from '../utils/helpers'
export default {
  data() {
    return {
      account: '',
      password: '',
      isProcessing: false,
      errorAccount: false,
      errorPassword: false
    }
  },
  methods: {
    async handleSubmit() {
      try {
        if (!this.account || !this.password) {
          Toast.fire({
            icon: 'warning',
            title: '請輸入帳號及密碼',
            background: '#FF6600',
            iconColor: '#fff',
            customClass: {
              container: 'sweetalert2-warning-pop',
            },
          })
          this.isInputNull = true
          return
        }


        const { data } = await adminAPI.signIn({
          account: this.account,
          password: this.password
        })
        this.isProcessing = true

        if (data.status === 'error') {
          throw new Error()
        }
        localStorage.setItem('token', data.token)
        this.$store.commit('setCurrentUser', data.user)       
        Toast.fire({
          icon: 'success',
          title: '登入成功！'
        })
        this.$router.push({name: "admin-tweets"})
      }catch(err){
        this.isProcessing = false
        let message = err.response.data.message

        if (message === "Error: admin permission denied") {
          Toast.fire({
            icon: 'error',
            title: '請由前台登入',
            background: '#FC5A5A',
            iconColor: '#fff',
            customClass: {
            container: 'sweetalert2-error-pop',
            },
          })
          this.errorAccount = true
        } else if (message === "Error: The account is incorrect!") {
          Toast.fire({
            icon: 'error',
            title: '帳號不存在！',
            background: '#FC5A5A',
            iconColor: '#fff',
            customClass: {
            container: 'sweetalert2-error-pop',
            },
          })
          this.errorAccount = true
        } else if (message === "Error: The password is incorrect!") {
          Toast.fire({
            icon: 'error',
            title: '密碼有誤',
            background: '#FC5A5A',
            iconColor: '#fff',
            customClass: {
            container: 'sweetalert2-error-pop',
            },
          })
          this.errorPassword = true
        }
      }
    }
  },
  watch: {
    account() {
      this.errorAccount = false
    },
    password() {
      this.errorPassword = false
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../assets/scss/admin-signIn-Up/style.scss';

</style>