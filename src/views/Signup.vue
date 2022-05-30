<template>
  <h1>Signup</h1>
  <div class="container">
    <form class="form" id="form">
      <span class="error">{{this.error}}</span>
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
      <input
        type="password"
        class="form-control"
        placeholder="Confirma la contraseña"
        id="password2"
      />
      <input type="text" class="form-control" placeholder="DNI" id="dni" />
      <input
        type="text"
        class="form-control"
        placeholder="Telefono"
        id="telefono"
      />
      <input type="email" class="form-control" placeholder="Email" id="email" />
      <input type="submit" class="btn btn-primary" @click="signUp()" />
      <input type="hidden" id="type" value="signup" />
    </form>
    <a href="/login">Ya tienes una cuenta? Inicia Sesión</a>
  </div>
</template>
<script>
export default {
  name: "Signup",
  data(){
    return {
      error : ""
    
    }
  },
  methods: {
    validate(username,password,password2,dni,telefono) {  
      function nif(dni) {
        var numero;
        var letr;
        var letra;
        var expresion_regular_dni;

        expresion_regular_dni = /^\d{8}[a-zA-Z]$/;

        if (expresion_regular_dni.test(dni) == true) {
          numero = dni.substr(0, dni.length - 1);
          letr = dni.substr(dni.length - 1, 1);
          numero = numero % 23;
          letra = "TRWAGMYFPDXBNJZSQVHLCKET";
          letra = letra.substring(numero, numero + 1);
          if (letra != letr.toUpperCase()) {
            this.error = "Dni erroneo";
            return false
          } else {
            return true
          }
        } else {
          this.error = "Dni erroneo";
          return false
        }
      }
      function passwordV(password,password2) {
        if(password == ""){
          this.error = "La contraseña no puede estar vacía"
          return false
        }else
        if(password.length < 8){
        this.error = "La contraseña no puede contener menos de 8 caracteres"
          return false
        }else

        if(password != password2){
          this.error = "Ambas contraseñas no coinciden";
          return false
        }else{
        return true;}

      }

      function telefonoV(telefono) {
         telefono = telefono.replace(/ /g,'')
        if(telefono.length != 9){
          this.error = "Introduce un numero de telefono valido"
          return false
        }else{
          return true
        }
      }

      function usernameV(username) {
        if(username == ""){
          this.error = "El nombre de usuario no puede estar vacío"
          return false
        }else if(username.length <5){
          this.error = "El nombre de usuario no puede tener menos de 5 caracteres"
          return false
        }else if(username.length >16){
          this.error = "El nombre de usuario no puede superar los 16 caracteres"
          return false
        }else{
          return true
        }
      }
      if(!nif(dni) || !passwordV(password,password2) || !telefonoV(telefono || !usernameV(username))){
        return false
      }else{
        return true
      }
      
    },
    signUp() {
      document.getElementById("form").addEventListener("submit", (e) => {
        e.preventDefault();
      });
      var username = document.getElementById("username").value.trim();
      var password = document.getElementById("password").value.trim();

      var password2 = document.getElementById("password2").value.trim();
      var dni = document.getElementById("dni").value.trim();
      //console.log(DNI)
      var telefono = document.getElementById("telefono").value.trim();
      var email = document.getElementById("email").value.trim();

      if (this.validate(username,password,password2,dni,telefono,email)) {
        var myHeaders = new Headers();
        myHeaders.append("Content-Type", "application/json");

        var raw = JSON.stringify({
          username: username,
          password: password,
          dni: dni,
          telf: telefono,
          email: email,
        });

        var requestOptions = {
          method: "POST",
          headers: myHeaders,
          body: raw,
          redirect: "follow",
        };

        fetch("http://127.0.0.1:8000/api/admins/register", requestOptions)
          .then((response) => response.text())
          .then((result) => console.log(result))
          .catch((error) => console.log("error", error));
      }else{
        alert("Arregle los errores");
      }
    },
  },
};
</script>
