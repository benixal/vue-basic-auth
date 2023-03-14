<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";

const email = ref("");
const password = ref("");
const message = ref("");
const loggedIn = ref(false);

function setCookie(name, value, days) {
  var expires = "";
  if (days) {
    var date = new Date();
    date.setTime(date.getTime() + days * 24 * 60 * 60 * 1000);
    expires = "; expires=" + date.toUTCString();
  }
  document.cookie = name + "=" + value + expires + "; path=/";
}

function getCookie(name) {
  var nameEQ = name + "=";
  var cookies = document.cookie.split(";");
  for (var i = 0; i < cookies.length; i++) {
    var cookie = cookies[i];
    while (cookie.charAt(0) == " ") cookie = cookie.substring(1, cookie.length);
    if (cookie.indexOf(nameEQ) == 0)
      return cookie.substring(nameEQ.length, cookie.length);
  }
  return null;
}

function deleteCookie(cookieName) {
  document.cookie = cookieName + "=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
}


onMounted(() => {
  axios
    .get("http://127.0.0.1:8000/api/user", {
      headers: {
        Authorization: "Bearer " + getCookie("token"),
      },
    })
    .then((res) => {
      message.value += " " + res.data.name;
      loggedIn.value = true;
    });
});

function logout() {
  axios
    .get("http://127.0.0.1:8000/api/logout", {
      headers: {
        Authorization: "Bearer " + getCookie("token"),
      },
    })
    .then((res) => {
      if (res.data) {
        message.value = "";
        loggedIn.value = false;
        deleteCookie("token");
      }
    });
}

function login() {
  axios
    .post("http://127.0.0.1:8000/api/login", {
      email: email.value,
      password: password.value,
    })
    .then((response) => {
      if (response.data.loggedin) {
        message.value = "welcome";

        setCookie("token", response.data.token, 30);
        axios
          .get("http://127.0.0.1:8000/api/user", {
            headers: {
              Authorization: "Bearer " + response.data.token,
            },
          })
          .then((res) => {
            message.value += " " + res.data.name;
            loggedIn.value = true;
          });
      } else {
        message.value = "wrong email or password";
      }
    });
}
</script>

<template>
  <div v-if="!loggedIn">
    <input type="text" placeholder="email" v-model="email" /><br />
    <input type="text" placeholder="password" v-model="password" /><br />

    <button @click="login">Login</button><br />
  </div>
  <div v-else>
    <button @click="logout">Logout</button>
  </div>

  <strong>{{ message }}</strong>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
