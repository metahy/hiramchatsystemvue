<template>
  <el-container>
    <el-aside v-if="this.useBy === 'pc'" width="200px" style="position: fixed; left: 0;  top: 0; height: 100%;">
      <div style="margin: 10px;"><span style="font-size: 14px;">当前在线用户</span></div>
      <el-card v-for="user in userList" :key="user" style="width: 80%; margin: 10px auto auto;">{{user}}</el-card>
    </el-aside>
    <el-container :style="{'margin-left':this.useBy === 'pc'?'200px':'0'}">
      <el-header style="width: 100%; height: 50px; padding-right: 200px; position: fixed; top: 0; line-height: 50px;">
        简单聊
      </el-header>
      <el-main id="chatContainer" style="height: calc(100vh - 170px); margin-top: 50px; padding-bottom: 10px">
        <div v-for="message in messages" :key="message.key">
          <el-alert v-if="message.type === 'system'"
                    :title=message.msg
                    type="info">
          </el-alert>
          <div v-if="message.type !== 'system' && message.nickName !== nickName"
               style="white-space:normal; display:inline;word-break:break-all; word-wrap:break-word;">
            <div style="margin: 2px 0 0 12px;">
              <time class="time" style="font-size: 8px;">{{message.nickName}}&nbsp;&nbsp;&nbsp;{{ message.time }}</time>
            </div>
            <el-card shadow="always"
                     style="max-width: 80%; margin: 0 10px 0 10px; width:auto; display: inline-block !important;">
              <span style="font-size: 14px;">{{message.msg}}</span>
            </el-card>
          </div>
          <div v-if="message.type !== 'system' && message.nickName === nickName"
               style="text-align: right; white-space:normal; display:inline;word-break:break-all; word-wrap:break-word;">
            <div style="margin: 2px 12px 0 0;">
              <time class="time" style="font-size: 8px;">{{message.time}}&nbsp;&nbsp;&nbsp;{{ message.nickName }}</time>
            </div>
            <el-card shadow="always"
                     style="float: right; clear: both; max-width: 80%; margin: 0 10px 0 10px; width:auto; display: inline-block !important;">
              <span style="font-size: 14px;">{{message.msg}}</span>
            </el-card>
          </div>
        </div>
      </el-main>
      <el-footer style="width: 100%; height: 120px; position: fixed; bottom: 0;"
                 :style="{'padding-right':this.useBy==='pc'?'220px':'20px'}">
        <el-input
          type="textarea"
          :rows="3"
          style="margin: 10px;"
          placeholder="请输入内容"
          @keyup.ctrl.enter.native="sendMessage"
          v-model="textarea">
        </el-input>
        <el-button type="primary" style="float: right; margin-top: -6px;" size="mini" @click="sendMessage">
          <el-icon name="el-icon-circle-check-outline"></el-icon>
          发送(Ctrl+Enter)
        </el-button>
      </el-footer>
    </el-container>
  </el-container>
</template>

<script>

  export default {
    data() {
      return {
        nickName: '',
        showSetNick: false,
        webSocket: null,
        textarea: '',
        messages: [],
        userList: [],
        useBy: '' // 手机或电脑
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
        if ((navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i))) {
          this.useBy = 'phone'
        } else {
          this.useBy = 'pc'
        }
      }
    },
    watch: {
      messages() {
        this.$nextTick(() => {
          var container = this.$el.querySelector("#chatContainer");
          container.scrollTop = container.scrollHeight;
        })
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
        let message = JSON.parse(e.data);
        console.log(message);
        if (message.type == 'setResult') {
          if (message.msg == -1) {
            this.nickName = '';
            vm.$message('昵称设置失败');
          } else {
            this.nickName = message.nickName;
            vm.nickName = message.nickName;
            vm.$message('昵称设置成功');
          }
        } else if (message.type == 'newUser') {
          console.log('new or lose');
          vm.userList = message.nickName;
          vm.$message({
            message: message.msg,
            type: 'success'
          });
        } else if (message.type == 'loseUser') {
          console.log('new or lose');
          vm.userList = message.nickName;
          vm.$message({
            message: message.msg,
            type: 'error'
          });
        } else if (message.type == 'showUser') {
          vm.userList = message.nickName;
        } else {
          vm.messages.push(message);
        }
      },
      webSocketClose() {  //关闭
        console.log("WebSocket关闭");
      },
      webSocketError() {  //失败
        console.log("WebSocket连接失败");
      },
      sendMessage() {
        let vm = this;
        if (vm.nickName.length <= 0) {
          this.open3();
          return;
        }
        if (vm.textarea && vm.textarea.length > 0) {
          let message = {};
          message.msg = vm.textarea;
          message.sendType = 'sendMsg';
          message.nickName = vm.nickName;
          this.finalSend(JSON.stringify(message));
          vm.textarea = null
        }
      },
      finalSend(message) {
        this.webSocket.send(message);
      },
      open3() {
        this.$prompt('输入你本次的昵称才能开始聊天', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          // inputPattern: /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/,
          // inputErrorMessage: '邮箱格式不正确'
        }).then(({value}) => {
          let body = {};
          body.msg = '';
          body.sendType = 'setNick';
          body.nickName = value;
          this.finalSend(JSON.stringify(body));
          this.$message({
            type: 'success',
            message: '你的昵称是: ' + value
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入将不能参与聊天'
          });
        });
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
