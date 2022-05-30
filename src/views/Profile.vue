<template>
    <Nav />
  <div>
    <h2>Datos personales</h2>
    <span>Nombre de Usuario : {{ this.user.username }}</span><br>
    <span>DNI : {{ this.user.dni }}</span><br>
    <span>Teléfono : {{ this.user.telf }}</span><br>
    <span>Modificar Contraseña</span><br>
  </div>
  <table>
    <tr>
      <td>Nombre</td>
      <td>Sector</td>
      <td>Borrar</td>
    </tr>
    <tr v-for="(company, index) in companies" v-bind:key="index">
      <td>{{ company.name }}</td>
      <td>{{ company.sector }}</td>
      <td>
        <button v-bind:id="index" @click="deleteCompany($event)">Borrar</button>
      </td>
    </tr>
  </table>
</template>

<script>
import Nav from "../components/Nav.vue"
export default {
  name: "Profile",
  components: {
      Nav
  },
  data() {
    return {
      companies: new Array(),
      user: new Object(),
    };
  },
  created() {

    if (localStorage.getItem("token") == undefined) {
      location.replace("/login");
    }

    this.fillCompanies();
    this.fetchData()
  },
  mounted() {

    var tag = document.createElement("link");
    tag.setAttribute("rel", "stylesheet");
    tag.setAttribute(
      "href",
      "https://unpkg.com/leaflet@1.8.0/dist/leaflet.css"
    );
    document.head.appendChild(tag);


     tag = document.createElement("link");
    tag.setAttribute("rel", "stylesheet");  
    tag.setAttribute("href", CSS);
    document.head.appendChild(tag);

    tag = document.createElement("script");
    tag.setAttribute(
      "src",
      "https://cdn.jsdelivr.net/gh/tomik23/autocomplete@1.8.3/dist/js/autocomplete.min.js"
    );
    document.head.appendChild(tag);

    

    tag = document.createElement("link");
    tag.setAttribute("rel", "stylesheet");
    tag.setAttribute(
      "href",
      "https://unpkg.com/bootstrap/dist/css/bootstrap.min.css"
    );
    document.head.appendChild(tag);

    tag = document.createElement("link");
    tag.setAttribute("rel", "stylesheet");
    tag.setAttribute(
      "href",
      "https://unpkg.com/bootstrap-vue@latest/dist/bootstrap-vue.min.css"
    );
    document.head.appendChild(tag);


    tag = document.createElement("link");
    tag.setAttribute("rel", "stylesheet");
    tag.setAttribute(
      "href",
      "https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,300,1,0"
    );
    document.head.appendChild(tag);
    },

  methods: {
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
            var c = new Company(
              json[i]["@id"],
              json[i]["name"],
              json[i]["sector"],
              json[i]["lat"],
              json[i]["lng"],
              json[i]["username"]
            );
            if (c.creator == localStorage.getItem("username")) {
              this.companies.push(c);
            }
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
            location.reload();
          })
          .catch((error) => console.log("error", error));
      }
    },
    fetchData() {
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

      fetch(
        "http://localhost:8000" + localStorage.getItem("username"),
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => {
          class User {
            constructor(username, dni, telf, email) {
              this.username = username;
              this.dni = dni;
              this.telf = telf;
              this.email = email;
            }
          }
          
          var json = JSON.parse(result);
          console.log(result);
          this.user = new User(
              json["username"],
              json["dni"],
              json["telf"],
              json["email"],
              );

              
        });
    },
  },
};
</script>