<template>
  <section class="container">
    <div>
      <img src="../assets/imgs/hiram.png"/>
      <el-button type="primary">haha</el-button>
    </div>
  </section>
</template>

<script>

export default {
  data () {
    return {
      webSocket: null,
      productinfos: {}
    }
  },
  created () {
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
  methods:{
    initWebSocket() {
      this.webSocket.onopen = this.webSocketOpen;
      this.webSocket.onmessage = this.webSocketMessage;
      this.webSocket.onclose = this.webSocketClose;
      this.webSocket.onerror = this.webSocketError;
    },
    webSocketOpen(){//打开
      console.log("WebSocket连接成功")
    },
    webSocketMessage(e){ //数据接收
      console.log(e.data);
    },
    webSocketClose(){  //关闭
      console.log("WebSocket关闭");
    },
    webSocketError(){  //失败
      console.log("WebSocket连接失败");
    },
  }
}
</script>

<style>

.container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
