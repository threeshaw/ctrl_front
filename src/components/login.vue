<!-- src/components/LockScreen.vue -->
<template>
  <div class="lock-screen">
    <h2>应用已锁定</h2>
    <form @submit.prevent="handleSubmit">
      <div>
        <label>用户名：</label>
        <input type="text" v-model="username" required />
      </div>
      <div>
        <label>密码：</label>
        <input type="password" v-model="password" required />
      </div>
      <button type="submit">解锁</button>
      <p v-if="error" class="error">{{ error }}</p>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      error: '',
    }
  },
  methods: {
    handleSubmit() {
      // 本地验证逻辑
      const isValid = this.verifyCredentials()

      if (isValid) {
        this.error = ''
        console.log(submit)
        this.$emit('unlock')
      } else {
        this.error = '用户名或密码错误'
      }
    },
    verifyCredentials() {
      // 从本地存储获取预存凭证
      const storedUsername = localStorage.getItem('app_username')
      const storedPassword = localStorage.getItem('app_password')

      // 实际应用中应对密码进行哈希处理，这里简化示例
      return this.username === storedUsername && this.password === storedPassword
    },
  },
}
</script>

<style scoped>
.lock-screen {
  max-width: 400px;
  margin: 100px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
}
.error {
  color: red;
}
</style>
