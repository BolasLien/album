<template lang="pug">
  #app
    b-navbar(toggleable="lg" type="dark" variant="primary")
      b-container
        b-navbar-brand(to='/') 相簿
        b-navbar-toggle(target='nav-collapse')
        b-collapse#nav-collapse(is-nav)
          b-navbar-nav.ml-auto
            b-nav-item(v-if="user.length === 0" to="/login") 登入
            b-nav-item(v-else to="/album") 我的相簿
            b-nav-item(v-if="user.length === 0" to="/reg") 註冊
            b-nav-item(v-else @click="logout") 登出
    b-container
      router-view
</template>

<script>
export default {
  name: 'app',
  computed: {
    user () {
      return this.$store.getters.user
    }
  },
  methods: {
    logout () {
      this.axios
        .delete(process.env.VUE_APP_APIURL + '/logout')
        .then(response => {
          const data = response.data
          // 如果回來的資料 success 是 true
          if (data.success) {
            alert('登出成功')
            // 呼叫 vuex 的登出
            this.$store.commit('logout')

            // 如果現在不是在首頁，跳到登出後的首頁
            if (this.$route.path !== '/') {
              // 跳到登出後的首頁
              this.$router.push('/')
            }
          } else {
            alert(data.message)
          }
        })
        .catch(error => {
          // 如果回來的狀態不是 200，顯示回來的 message
          alert(error.response.data.message)
        })
    },
    heartbeat () {
      this.axios
        .get(process.env.VUE_APP_APIURL + '/heartbeat')
        .then(response => {
          const data = response.data

          if (this.user.length > 0) {
            if (!data) {
              alert('登入時效已過')
              // 前端登出
              this.$store.commit('logout')
              // 如果現在不是在首頁，跳到登出後的首頁
              if (this.$route.path !== '/') {
                this.$router.push('/')
              }
            }
          }
        })
        .catch(() => {
          alert('發生錯誤')
          this.$store.commit('logout')
          // 如果現在不是在首頁，跳到登出後的首頁
          if (this.$route.path !== '/') {
            this.$router.push('/')
          }
        })
    }
  },
  mounted () {
    this.heartbeat()
    setInterval(() => {
      this.heartbeat()
    }, 1000 * 5)
  }
}
</script>
