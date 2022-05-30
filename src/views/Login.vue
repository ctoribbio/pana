<template>
  <h1 @load="load()">Login</h1>
  <div class="container">
    <form class="form" id="form">
      <h4 id="error"></h4>
      <input
        type="text"
        class="form-control"
        placeholder="Username"
        id="username"
      />
      <input
        type="password"
        class="form-control"
        placeholder="Password"
        id="password"
      />
      <input type="submit" class="btn btn-primary" @click="login()" />
      <input type="hidden" id="type" value="login" />
    </form>

    <a href="/signup">Aún no estas registrado? Reigstrate</a>
  </div>
</template>
<script>
export default {
  name: "Login",
  created() {
    localStorage.clear();
  },
  data() {
    return {};
  },
  methods: {
    login() {
      document.forms[0].addEventListener("submit", (e) => {
        e.preventDefault();
      });
      var username = document.getElementById("username").value.trim();
      var password = document.getElementById("password").value.trim();

      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");

      var raw = JSON.stringify({
        username: username,
        password: password,
      });

      var requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: raw,
        redirect: "follow",
      };

      var failure = {
        value: false,
        toString() {
          return this.value;
        },
        setValue(val) {
          this.value = val;
        },
      };

      fetch("http://localhost:8000/api/login_check", requestOptions)
        .then((response) => response.text())
        .then((result) => {
          if (JSON.parse(result)["token"] != undefined) {
            this.getRole(username);
            localStorage.setItem("token", JSON.parse(result)["token"]);
            localStorage.setItem("username", this.getIri(username));
          } else {
            document.getElementById("error").innerText =
              "Usuario o contraseña incorrectas";
            failure.setValue(true);
          }
          if (failure != true) {
            setTimeout(function () {
              if (localStorage.getItem("role") == "ROLE_USER") {
                location.replace("http://127.0.0.1:8080/main");
              } else {
                location.replace("http://127.0.0.1:8080/admin");
              }
            }, 1000);
          }
        })

        .catch(() => failure.setValue(true));
    },
    inputIri(data) {
      return JSON.parse(data)["hydra:member"][0]["@id"];
    },
    getRole(username) {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/ld+json");

      var requestOptions = {
        method: "GET",
        headers: myHeaders,
        redirect: "follow",
      };

      fetch(
        "http://localhost:8000/api/admins?username=" + username,
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => {
          var json = JSON.parse(result);
          if (json["hydra:member"][0] != undefined) {
            localStorage.setItem("role", json["hydra:member"][0]["roles"][0]);
          }
        })
        .catch((error) => console.log("error", error));
    },
    getIri(username) {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/ld+json");

      var requestOptions = {
        method: "GET",
        headers: myHeaders,
        redirect: "follow",
      };

      fetch(
        "http://localhost:8000/api/admins?username=" + username,
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => {
          var data = this.inputIri(result);
          localStorage.setItem("username", data);
        })
        .catch((error) => console.log("error", error));
    },
  },
};
</script>
