<script setup lang="ts">
// Types
interface InputForm {
  email: string;
  password: string;
}

interface SendMessage {
  data: string | InputForm;
  command: "open" | "data";
}

const allowOrigins = [
  "http://127.0.0.1:5174",
  "https://codeliners-post-message-window-a.netlify.app",
];
let targetOrigin: string | undefined = undefined;
let sourceOrigin: MessageEventSource | null = null;

const form = {
  email: "",
  password: "",
};

window.addEventListener("message", receiveMessage, false);

function receiveMessage(event: MessageEvent) {
  targetOrigin = allowOrigins.find((origin) => origin === event.origin);
  if (!targetOrigin) return;
  const { data } = event;
  const parsedData = JSON.parse(data) as SendMessage;

  if (parsedData.command === "open") {
    sourceOrigin = event.source;

    sendMessage({ data: "opened", command: "open" });
    return;
  }
}

function sendMessage({ data, command }: SendMessage) {
  if (!sourceOrigin) return;

  sourceOrigin.postMessage(JSON.stringify({ data, command }), {
    targetOrigin,
  });
}

async function onSubmit() {
  // Has source origin?
  if (!sourceOrigin) return;

  // Send message to opened origin.
  sendMessage({ data: form, command: "data" });

  // Close window.
  window.close();
}
</script>

<template>
  <div>
    <form class="form" @submit.prevent="onSubmit">
      <input type="text" placeholder="Email" v-model="form.email" />
      <input type="password" placeholder="Password" v-model="form.password" />
      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<style scoped>
.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>
