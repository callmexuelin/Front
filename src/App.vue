<template>
  <div class="app">
    <!-- 输入框，用于手动输入问题 -->
    <input v-model="userQuery" placeholder="请输入您的问题" class="input-box" />
    
    <!-- 点击按钮后发送请求 -->
    <button @click="getChatMessage">发送</button>
  </div>
</template>

<script>
import { onMounted } from 'vue';
import * as live2d from 'live2d-render';

export default {
  data() {
    return {
      userQuery: '',    // 存储用户输入的问题
    };
  },
setup(){
  // 初始化live2d模型
  onMounted(async () => {
    await live2d.initializeLive2D({
        // background color of your live2d canvas 
        BackgroundRGBA: [0.0, 0.0, 0.0, 0.0],

        // path relative to your index.html
        // must be a *.model3.json
        ResourcesPath: './jingliu/jingliu.model3.json',
        
        // height and width of the live2d model
        CanvasSize: {
            height: 500,
            width: 400
        },

        ShowToolBox: true,
        
        // If `LoadFromCache` is set as true, all the static resources
        // like texture and model3.json will be stored in indexDB
        // as a quick cache
        LoadFromCache: true
    });
    console.log('finish loading');
  });
  },
  methods:{
    // 主函数逻辑
    async getChatMessage() {
      try {
        const res = await fetch('http://127.0.0.1:5000/ask', {
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
        console.log(data.answer);
        // updateLive2DDialog(this.response.answer)
        live2d.setMessageBox(this.response.answer, 3000);
        console.log(live2d)
      } catch (error) {
        console.error('Error:', error);
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
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#live2dMessageBox-content {
    background-color: #FF95BC;
    color: white;
    font-family: var(--base-font);
    padding: 10px;
    height: fit-content;
    border-radius: .7em;
    word-break: break-all;
    border-right: 1px solid transparent;
}

.live2dMessageBox-content-hidden {
    opacity: 0;
    transform: scaleY(0.2);
    transition: all 0.35s ease-in;
    -moz-transition: all 0.35s ease-in;
    -webkit-transition: all 0.35s ease-in;
}

.live2dMessageBox-content-visible {
    opacity: 1;
    transition: all 0.35s ease-out;
    -moz-transition: all 0.35s ease-out;
    -webkit-transition: all 0.35s ease-out;
}

</style>
