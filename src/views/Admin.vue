<template>
  <div class="list">
    <h2>Users</h2>
    <table class="table">
      <tr>
        <th>Username</th>
        <th>DNI</th>
        <th>Telefono</th>
        <th>E-Mail</th>
        <th>Ha formalizado su empresa</th>
        <th>Borrar</th>
      </tr>
      <tr v-for="(user, index) in users" v-bind:key="index">
        <td>{{ user.username }}</td>
        <td>{{ user.dni }}</td>
        <td>{{ user.telf }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.bool }}</td>
        <td>
          <button v-bind:id="index" @click="deleteUser($event)">Borrar</button>
        </td>
      </tr>
    </table>
    <h2>Companies</h2>
    <table class="table">
      <tr>
        <th>Nombre</th>
        <th>Sector</th>
        <th>Latitud</th>
        <th>Logitud</th>
        <th>Creador</th>
        <th>Borrar</th>
      </tr>
      <tr v-for="(company, index) in companies" v-bind:key="index">
        <td>{{ company.name }}</td>
        <td>{{ company.sector }}</td>
        <td>{{ company.lat }}</td>
        <td>{{ company.lng }}</td>
        <td>{{ company.creator }}</td>
        <td>
          <button v-bind:id="index" @click="deleteCompany($event)">
            Borrar
          </button>
        </td>
      </tr>
    </table>
  </div>
  <a href="/login">Cerrar Sesion</a>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      users: new Array(),
      companies: new Array(),
    };
  },
  created() {
    if (localStorage.getItem("token") == undefined) {
      location.replace("/login");
    }

    this.fillUsers();
    this.fillCompanies();
  },
  methods: {
    fillUsers() {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/ld+json");

      var requestOptions = {
        method: "GET",
        headers: myHeaders,
        redirect: "follow",
      };

      fetch("http://localhost:8000/api/admins", requestOptions)
        .then((response) => response.text())
        .then((result) => {
          class Admin {
            constructor(iri, username, dni, telf, email, bool) {
              this.iri = iri;
              this.username = username;
              this.dni = dni;
              this.username = username;
              this.telf = telf;
              this.email = email;
              this.bool = bool;
            }
          }

          var json = JSON.parse(result)["hydra:member"];
          for (var i = 0; i < json.length; i++) {
            console.log("a");
            var a = new Admin(
              json[i]["@id"],
              json[i]["username"],
              json[i]["dni"],
              json[i]["telf"],
              json[i]["email"],
              json[i]["companyAdded"]
            );
            this.users.push(a);
          }
        })
        .catch((error) => console.log("error", error));
    },
    fillCompanies() {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/ld+json");
      myHeaders.append(
        "Authorization",
        "Bearer " + localStorage.getItem("token")
      );

      var requestOptions = {
        method: "GET",
        headers: myHeaders,
        redirect: "follow",
      };

      fetch("http://localhost:8000/api/companies", requestOptions)
        .then((response) => response.text())
        .then((result) => {
          class Company {
            constructor(iri, name, sector, lat, lng, creator) {
              this.iri = iri;
              this.name = name;
              this.sector = sector;
              this.lat = lat;
              this.lng = lng;
              this.creator = creator;
            }
          }
          var json = JSON.parse(result)["hydra:member"];
          for (var i = 0; i < json.length; i++) {
            var x;
            for (var j = 0; j < this.users.length; j++) {
              if (this.users[j].iri == json[i]["username"]) {
                x = this.users[j].username;
                break;
              }
            }
            var c = new Company(
              json[i]["@id"],
              json[i]["name"],
              json[i]["sector"],
              json[i]["lat"],
              json[i]["lng"],
              x
            );
            this.companies.push(c);
          }
        })
        .catch((error) => console.log("error", error));
    },
    deleteCompany(event) {
      if (confirm("Are you sure you want to delete")) {
        var iri = this.companies[event.target.id].iri;
        console.log(iri);
        var myHeaders = new Headers();
        myHeaders.append(
          "Authorization",
          "Bearer " + localStorage.getItem("token")
        );

        var requestOptions = {
          method: "DELETE",
          headers: myHeaders,
          redirect: "follow",
        };

        fetch("http://localhost:8000" + iri, requestOptions)
          .then((response) => response.text())
          .then((result) => {
            console.log(result);
            location.reload()
          })
          .catch((error) => console.log("error", error));
      }
    },
    deleteUser(event) {
      if (confirm("Are you sure you want to delete")) {
        var iri = this.users[event.target.id].iri;
        var myHeaders = new Headers();
        myHeaders.append(
          "Authorization",
          "Bearer" + localStorage.getItem("token")
        );

        var requestOptions = {
          method: "DELETE",
          headers: myHeaders,
          redirect: "follow",
        };

        fetch("http://localhost:8000" + iri, requestOptions)
          .then((response) => response.text())
          .then((result) => {
            console.log(result);
            location.reload();
          })
          .catch((error) => console.log("error", error));
      }
    },
  },
};
</script>