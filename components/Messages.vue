<template>
  <div>
    <h1>Messages</h1>
    <div v-for="entry in messages"><span>{{entry.username}}</span><span>{{entry.message}}</span></div>
  </div>
</template>

<script>
import {IdentitySerializer, JsonSerializer, RSocketClient} from "rsocket-core";
import RSocketWebSocketClient from "rsocket-websocket-client/build";

export default {
  name: "Messages",
  data() {
    return {
      messages: [],
    }
  },
  methods: {
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
    connectStream() {
      this.client.connect().subscribe({
        onComplete: (socket) => {
          if (socket) {
            this.socket = socket;
            const payload = {username: 'user2', message: 'Joining'};
            socket
              .requestStream({
                data: payload,
                metadata:
                  String.fromCharCode("subscribe".length) +
                  "subscribe",
              })
              .subscribe({
                onComplete: (data) => {
                  console.log("got response with requestResponse", data);
                },
                onError: (error) => {
                  console.log("got error with requestResponse");
                  console.error(error);
                },
                onNext: (payload) => {
                  console.log("onNext", payload);
                  this.messages.push(payload.data)
                },
                onSubscribe: (subscription) => {
                  console.log(subscription);
                  subscription.request(2)
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
  },
  async created() {
    await this.setupConn()
    this.connectStream()
  }
}
</script>
