<template>
  <el-container>
    <el-aside width="200px" style="position: fixed; left: 0;  top: 0; height: 100%;">Aside</el-aside>
    <el-container style="margin-left: 200px;">
      <el-header style="width: 100%; height: 50px; padding-right: 200px; position: fixed; top: 0;">Header</el-header>
        <el-main style="height: calc(100vh - 150px); margin-top: 50px; margin-bottom: 60px;">
          <div v-for="message in messages" :key="message.key">
            <div v-if="message.type === 'notice'">
              <span style="font-size: 8px; margin: 2px 0 0 12px; color: #47494e;">{{message.msg}}</span>
            </div>
            <div v-if="message.type !== 'notice'" style="white-space:normal; display:inline;word-break:break-all; word-wrap:break-word;">
              <div style="margin: 2px 0 0 12px;">
                <time class="time" style="font-size: 8px;">{{ message.time }}</time>
              </div>
              <el-card shadow="always" style="max-width: 80%; margin: 0 10px 0 10px; width:auto; display: inline-block !important;">
                <span style="font-size: 14px;">{{message.msg}}</span>
              </el-card>
            </div>
          </div>
        </el-main>
      <el-footer style="width: 100%; height: 120px; padding-right: 220px; position: fixed; bottom: 0;">
        <el-input
          type="textarea"
          :rows="3"
          style="margin: 10px;"
          placeholder="请输入内容"
          v-model="textarea">
        </el-input>
        <el-button type="primary" style="float: right; margin-top: -6px;" size="mini" @click="sendMessage">
          <el-icon name="el-icon-circle-check-outline"></el-icon>发送
        </el-button>
      </el-footer>
    </el-container>
  </el-container>
</template>

<script>

  export default {
    data() {
      return {
        webSocket: null,
        textarea: '',
        messages: []
      }
    },
    created() {
      if (process.browser) {
        if ('WebSocket' in window) {
          this.webSocket = new WebSocket("ws://localhost:8080/websocket");
          this.initWebSocket();
          console.log('Support webSocket')
        }
        else {
          console.log('Not support webSocket')
        }
      }
    },
    methods: {
      initWebSocket() {
        this.webSocket.onopen = this.webSocketOpen;
        this.webSocket.onmessage = this.webSocketMessage;
        this.webSocket.onclose = this.webSocketClose;
        this.webSocket.onerror = this.webSocketError;
      },
      webSocketOpen() {//打开
        console.log("WebSocket连接成功")
      },
      webSocketMessage(e) { //数据接收
        let vm = this;
        console.log(e);
        let message = JSON.parse(e.data);
        vm.messages.push(message)
      },
      webSocketClose() {  //关闭
        console.log("WebSocket关闭");
      },
      webSocketError() {  //失败
        console.log("WebSocket连接失败");
      },
      sendMessage() {  //失败
        let vm = this;
        if (vm.textarea && vm.textarea.length>0) {
          this.webSocket.send(vm.textarea);
          vm.textarea = null
        }
      }
    }
  }
</script>

<style>
  .el-header, .el-footer {
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    padding: 0 0;
  }
  .el-aside {
    background-color: #D3DCE6;
    color: #333;
    text-align: center;
  }
  .el-main {
    padding: 0 0;
  }
  .el-card__body {
    padding: 5px 10px 5px;
    line-height: 16px;
  }
  body > .el-container {
    margin-bottom: 40px;
  }
</style>
