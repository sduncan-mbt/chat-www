<template>
  <div>
    <input type="text" v-model="username" placeholder="Username">
    <input type="text" v-model="message" placeholder="Message">
    <button @click="sendMessage" class="bg-green">Send Message</button>
  </div>
</template>

<script>
import {IdentitySerializer, JsonSerializer, RSocketClient} from "rsocket-core";
import RSocketWebSocketClient from "rsocket-websocket-client/build";

export default {
  name: "SendMessage",
  data() {
    return {
      username: '',
      message: '',
    }
  },
  methods: {
    sendMessage() {
      this.client.connect().subscribe({
        onComplete: (socket) => {
          if (socket) {
            this.socket = socket;
            const payload = {username: this.username, message: this.message};
            socket
              .requestResponse({
                data: payload,
                metadata:
                  String.fromCharCode("publish".length) +
                  "publish",
              })
              .subscribe({
                onComplete: (data) => {
                  console.log("got response with requestResponse", data);
                },
                onError: (error) => {
                  console.log("got error with requestResponse");
                  console.error(error);
                },
                onSubscribe: (subscription) => {
                  console.log(subscription);
                  /* call cancel() to stop onComplete/onError */
                },
              });
          } else {
            console.log("not connected...");
          }
        },
        onError: (error) => {
          console.log("got error");
          console.error(error);
        },
        onSubscribe: (cancel) => {
          /* call cancel() to abort */
          console.log("subscribe!");
          this.question = '';
          console.log(cancel);
          // cancel.cancel();
        },
      });
    },
    setupConn() {
      this.client = new RSocketClient({
        serializers: {
          data: JsonSerializer,
          metadata: IdentitySerializer,
        },
        setup: {
          keepAlive: 60000,
          lifetime: 180000,
          dataMimeType: "application/json",
          metadataMimeType: "message/x.rsocket.routing.v0",
        },
        transport: new RSocketWebSocketClient({
          url: "ws://localhost:12342/rs",
        }),
      });
    },
  },
  created() {
    this.setupConn()
  }
}
</script>
