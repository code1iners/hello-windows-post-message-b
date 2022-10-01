<script setup lang="ts">
import { ref } from "vue";

// Types
interface InputForm {
  email: string;
  password: string;
}

interface SendMessage {
  data: string | InputForm;
  command: "open" | "data";
}

// Variables.
const allowOrigins = [
  "http://127.0.0.1:5555",
  "https://codeliners-post-message-window-a.netlify.app",
];
let targetOrigin: string | undefined = undefined;
let sourceOrigin: MessageEventSource | null = null;
let isPageInvalid = ref(false);

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
  if (!sourceOrigin) {
    alert("This page is invalid.\nPlease try again valid way.");
    isPageInvalid.value = true;
    return;
  }

  // Send message to opened origin.
  sendMessage({ data: form, command: "data" });

  // Close window.
  window.close();
}
</script>

<template>
  <div>
    <p v-if="isPageInvalid">This page is invalid.</p>
    <form v-else class="form" @submit.prevent="onSubmit">
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
