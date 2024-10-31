<template>
    <div class="login-container">
      <h2>登录</h2>
      <input v-model="username" placeholder="账号" class="input" />
      <input type="password" v-model="password" placeholder="密码" class="input" />
      <button @click="login" class="login-button">登录</button>
      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        username: '',
        password: '',
        errorMessage: ''
      };
    },
    methods: {
      async login() {
        try {
          const res = await fetch('http://127.0.0.1:5000/login', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({
              username: this.username,
              password: this.password
            })
          });
          const data = await res.json();
          
          if (data.success) {
            // 登录成功，存储用户信息并跳转
            localStorage.setItem('username', this.username);
            this.$router.push('/chat'); // 跳转到聊天页面 (假设路径是/chat)
          } else {
            // 显示错误信息
            this.errorMessage = data.message || '登录失败，请重试。';
          }
        } catch (error) {
          console.error('Error:', error);
          this.errorMessage = '登录失败，请稍后再试。';
        }
      }
    }
  }
  </script>
  
  <style>
  .login-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    background-color: #f7f7f7;
  }
  
  .input {
    margin: 10px;
    padding: 10px;
    width: 200px;
  }
  
  .login-button {
    padding: 10px 15px;
    cursor: pointer;
  }
  
  .error {
    color: red;
  }
  </style>
  