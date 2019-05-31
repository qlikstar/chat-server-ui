<template>
    <div id="app">
        <div id="main-content" class="container">
            <div class="row">
                <div class="col-md-6">
                    <form class="form-inline">
                        <div class="form-group">
                            <label for="name">What is your name?</label>
                            <input type="text" id="name" class="form-control" v-model="name" placeholder="Your name here...">
                        </div>
                        <button id="send" class="btn btn-default" variant="dark" type="submit" @click.prevent="send">Send</button>
                    </form>
                </div>
            </div>
            <div class="row">
                <div class="col-md-12">
                    <h2>Greetings</h2>
                    <table id="conversation" >
                        <tbody>
                            <tr v-for="item in received_messages" :key="item">
                                <td>{{ item }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import SockJS from 'sockjs-client'
import Stomp from 'webstomp-client'

export default {
  name: 'app',
  data () {
    return {
      received_messages: [],
      name: null,
      connected: false
    }
  },
  methods: {
    send () {
      console.log('Send message:' + this.name)
      if (this.stompClient && this.stompClient.connected) {
        const msg = { from: this.name,
                      text: "Something nice"
                    }
        this.stompClient.send('/app/chat/hello', JSON.stringify(msg), {})
      }
    },
    connect () {
      this.socket = new SockJS('http://localhost:9090/chat')
      this.stompClient = Stomp.over(this.socket)
      this.stompClient.connect(
        {},
        frame => {
          this.connected = true
          console.log(frame)
          this.stompClient.subscribe('/topic/messages', tick => {
            console.log(tick.body)
            // if (this.name !== JSON.parse(tick.body)["from"]) {
              this.received_messages.push(JSON.parse(tick.body))
            // }
          })
        },
        error => {
          console.log(error)
          this.connected = false
        }
      )
    },
    disconnect () {
      if (this.stompClient) {
        this.stompClient.disconnect()
      }
      this.connected = false
    },
    tickleConnection () {
      this.connected ? this.disconnect() : this.connect()
    }
  },
  mounted () {
    this.connect()
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
