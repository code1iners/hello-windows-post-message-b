<script setup lang="ts">
import { ref } from "vue";
import { usePopup, SendMessage } from "@ce1pers/window-helpers";

const {
  sendMessageToSourceOrigin,
  setSourceOrigin,
  getSourceOrigin,
  setTargetOrigin,
  getTargetOrigin,
} = usePopup({
  onMessageCallback,
});

// Variables.
const allowOrigins = [
  "http://127.0.0.1:5555",
  "http://localhost:5555",
  "https://codeliners-post-message-window-a.netlify.app",
];

const isPageInvalid = ref(false);
const isReady = ref(false);

const form = {
  email: "",
  password: "",
};

function onMessageCallback(event: MessageEvent) {
  const targetOrigin = allowOrigins.find((origin) => origin === event.origin);
  if (!targetOrigin) return;
  // Set target origin for send post message.
  setTargetOrigin(targetOrigin);

  const { data, source } = event;
  const parsedData = JSON.parse(data) as SendMessage;

  switch (parsedData.type) {
    case "open":
      if (source) {
        // Set source origin.
        setSourceOrigin(source);
        // Send opened signal.
        sendMessageToSourceOrigin({ data: "The window opened.", type: "open" });
      }
      break;

    case "ready":
      // Set ready status.
      isReady.value = true;
      break;
  }
}

async function onSubmit() {
  // Has source origin?
  if (!getSourceOrigin() || !getTargetOrigin()) {
    alert("This page is invalid.\nPlease try again valid way.");
    isPageInvalid.value = true;
    return;
  }

  // Send message to opened origin.
  sendMessageToSourceOrigin({ type: "data", data: form });

  // Close window.
  window.close();
}
</script>

<template>
  <div>
    <div v-if="isReady">
      <p v-if="isPageInvalid">This page is invalid.</p>
      <article class="form__container" v-else>
        <h3>Sample Login Form</h3>
        <form class="form" @submit.prevent="onSubmit">
          <input type="text" placeholder="Email" v-model="form.email" />
          <input
            type="password"
            placeholder="Password"
            v-model="form.password"
          />
          <button type="submit">Submit</button>
        </form>
      </article>
    </div>
    <div v-else>Waiting...</div>
  </div>
</template>

<style scoped>
.form__container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.form input {
  padding: 10px;
}
</style>
