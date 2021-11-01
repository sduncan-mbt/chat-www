<template>
  <div>
    <send-message/>
    <messages/>
  </div>
</template>

<script>
import SendMessage from "../components/SendMessage";
import Messages from "../components/Messages";

export default {
  components: {Messages, SendMessage}
}
</script>
