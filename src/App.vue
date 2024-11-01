<template>
  <div class="app">
    <!-- 背景内容区 -->
    <div class="background-answer">
      <div v-for="(msg, index) in chatHistory" :key="index" class="chat-message" :class="{'user-message-bg': msg.user === '你', 'robot-message-bg': msg.user === '机器人'}">
        <span class="user-label">{{ msg.user }}: </span>{{ msg.text }}
      </div>
    </div>
    
    <!-- 输入框容器 -->
    <div class="input-container">
      <input 
        v-model="userQuery" 
        placeholder="请输入您的问题" 
        class="input-box" 
        @keypress.enter="getChatMessage" 
      />
      
      <!-- 点击按钮后发送请求 -->
      <button @click="getChatMessage" class="send-button">发送</button>
    </div>
  </div>
</template>

<script>
import { onMounted } from 'vue';
import * as live2d from 'live2d-render';

export default {
  data() {
    return {
      userQuery: '',     // 存储用户输入的问题
      response: {},      // 存储后台返回的响应
      chatHistory: []     // 存储聊天记录
    };
  },
  setup() {
    // 初始化live2d模型
    onMounted(async () => {
      await live2d.initializeLive2D({
        BackgroundRGBA: [0.0, 0.0, 0.0, 0.0],
        ResourcesPath: './hiyori_free_en/runtime/hiyori_free_t08.model3.json',
        CanvasSize: {
          height: 500,
          width: 400
        },
        ShowToolBox: true,
        LoadFromCache: true
      });
      console.log('finish loading');
    });
  },
  methods: {
    // 主函数逻辑
    async getChatMessage() {
      if (this.userQuery.trim() === '') return; // 如果输入为空，则不发送

      // 判断用户输入是否为 "clear"
      if (this.userQuery.trim().toLowerCase() === 'clear') {
      this.chatHistory = []; // 清空聊天记录
      this.userQuery = ''; // 清空输入框
      return; // 早退，不发送任何消息
      }


      // 先将用户输入的消息添加到聊天记录中
      this.chatHistory.push({ user: '你', text: this.userQuery });
      
      try {
        const res = await fetch('http://localhost:5000/ask', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            query: this.userQuery,
            user: 'abc-123', // 假设这个是固定的用户ID
            conversation_id: '' // 假设这是一个新的对话
          })
        });
        const data = await res.json();
        this.response = data;

        // 将后台返回的信息也添加到聊天记录中
        this.chatHistory.push({ user: '机器人', text: data.answer });
        live2d.setMessageBox(data.answer, 3000);

        
        console.log(data.answer);
      } catch (error) {
        console.error('Error:', error);
      } finally {
        // 清空输入框
        this.userQuery = '';
      }
    },
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left; /* 将文本对齐方式改为左对齐 */
  color: #2c3e50;
  margin-top: 60px;
  position: relative;
  height: calc(100vh - 60px); /* Fill the viewport height, minus space for input */
}

.background-answer {
  position: fixed;
  top: 0; /* 保持顶部固定 */
  left: 0;
  right: 0;
  bottom: 60px; /* 确保输入框有空间，不被遮挡 */
  background-color: rgba(255, 255, 255, 0.5); /* 背景色 */
  color: white;
  display: flex;
  flex-direction: column; 
  justify-content: flex-start; 
  padding: 20px;
  overflow-y: auto; /* 允许垂直滚动 */
}

.chat-message {
  margin-bottom: 10px;
  padding: 10px; /* 添加内边距 */
  border-radius: 5px; /* 边角圆润 */
}

.user-message-bg {
  background-color: rgba(0, 123, 255, 0.7); /* 用户消息背景色 */
  color: white;
  text-align: left; /* 左对齐 */
}

.robot-message-bg {
  background-color: rgba(255, 149, 188, 0.7); /* 机器人消息背景色 */
  color: black;
  text-align: left; /* 左对齐 */
}

.user-label {
  font-weight: bold; /* 用户标签加粗 */
}

.input-container {
  position: fixed; 
  bottom: 0; 
  right: 0; /* 确保输入框和按钮在最右边 */
  z-index: 2; /* 确保输入框在背景之上 */
  display: flex; 
  align-items: center; 
  padding: 10px; 
}

.input-box {
  width: 400px; /* 设置输入框宽度 */
  padding: 10px; /* 输入框内边距 */
  margin-right: 10px; /* 输入框与按钮之间的间距 */
}

.send-button {
  padding: 10px 15px; /* 按钮内边距 */
}

/* 其他样式保持不变 */
</style>
