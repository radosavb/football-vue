<template>
  <div class="admin-panel">
   
<!-- LogIn formular -->
<form  class="login-form" @submit.prevent="autorizacija">
  <tr v-show="!isAuthorized ==='true'">
    <td><label  for="username_login" ><strong>Username:</strong> </label></td>
    <td><input id="username_login" type="text" placeholder="Unesite username" v-model="username"/></td>
  </tr>
  <tr v-show="!isAuthorized ==='true'">
    <td> <label for="password_login" ><strong>Password:</strong> </label></td>
    <td><input id="password_login" type="password" placeholder="Unesite sifru" v-model="password"/></td>
  </tr>
      <!-- User Panel -->
      <span class="submit-wrapper">
          <input v-if="!isAuthorized==='true'" type="submit" value="Log In"/>
          <div v-else><span class="ml-1 mt-2" id="pozdrav">Welcome {{username_show}}</span><span class="slicica-profil" :class="username_show"></span>
            <input  @click="otkazivanje()" class=" bg-danger ml-1 mt-1"  type="button" value="Otkazi igru" v-show="isAuthorized ==='true' && !otkazani.some(e => e.username === username_show) "/>
            <input v-show="isAuthorized ==='true' && otkazani.some(e => e.username === username_show) && !players.some(e => e.name === username_show)" class="bg-primary mt-1 ml-1" @click="ponovnaPrijava()" type="button" value="Vrati se u neodredjene" /> 
            <input class="bg-info ml-1 mt-1"  @click="logout()" type="button" value="Log Out"/>   
              <hr>
            <form @submit.prevent="promenaSifre()" class="promeni-sifru" style="display: inline-block">
              <input v-model="nova_sifra" id="promeni sifru" type="password" placeholder="Unesi novu sifru"/>{{" "}}
              <input v-model="provera_sifre" id="ponovi sifru" type="password" placeholder="Ponovi novu sifru"/>{{" "}}
              <input type="submit" value="Promeni sifru"/>
            </form>
          </div>
      </span>
      <!-- Kraj User Panela -->
  </form>
<!-- Kraj LogIn formulara -->
<div v-show="greska_prijava" class=" zauzeto alert alert-warning alert-dismissible fade show mt-2" role="alert">
    <strong>GRESKA!</strong> {{greska_prijava}}
    <button @click="greska_prijava=''" type="button" class="close" data-dismiss="alert" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>
  </div>
<!-- Dugmad za otkazivanje drugara -->
  <div>
   <span v-show="drugar.username == username_show" v-for="drugar in drugari" :key="drugar.drugar"><button @click="[promeniDrugara(drugar),otkaziDrugara()]" class="btn btn-danger p-1 mr-1 mb-1">Otkazi za {{drugar.drugar}}</button></span>
 </div>

<!-- Formular za admine -->
  <form class="login-form admin-form" @submit.prevent="add_user" v-show="isAuthorized === 'true' && admins.includes(username_show)">
    <h5>Dodaj novog korisnika</h5>
    <tr >
      <td><label for="username" >Username novog korisnika: </label></td>
      <td><input id="username" type="text" placeholder="Unesite username" v-model="new_user_username"/></td>
    </tr>
    <tr >
      <td> <label for="password" >Password novog korisnika: </label></td>
      <td><input id="password" type="password" placeholder="Unesite sifru" v-model="new_user_password"/></td>
    </tr>
      <span class="submit-wrapper">
          <input  type="submit" value="Add User"/>
      </span>
  </form>
<!-- kraj formulara za admine -->

<!-- Alert za promenu sifre -->
<div v-show="isAuthorized === 'true' && username_show" class="admin-panel-list"> 

  <div v-show="obavestenje_o_promeni" class=" zauzeto alert alert-warning alert-dismissible fade show" role="alert">
    <strong>USPEH!</strong> {{obavestenje_o_promeni}}
    <button @click="obavestenje_o_promeni=''" type="button" class="close" data-dismiss="alert" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>
  </div>

<!-- Lista svih igraca -->
 <h6>LOBBY:</h6> 
  <ul>
    <li v-for="user in users" :key="user.username">
      <span :class="{otkazao: otkazani.some(e => e.username === user.username), prijavljen: players.some(player => player.name == user.username)}" class="status"></span>
      {{user.username}}
      <span class="slicica" :class="user.username"></span>
    </li>
  </ul>

  <ul><li v-show="!users.some(e => e.username == player.name)" v-for="player in players" :key="player.name"><span class="status prijavljen"></span>{{player.name}}<span class="slicica default"></span></li></ul>
</div>
</div>
</template>

<script>

export default {
  name: 'AdminPanel',
  
  data(){
    return{
      users: [],
      players:[],
      otkazani:[],
      drugari:[],
      admins:['radosavb'],
      username: "",
      username_show: localStorage.getItem("username"),
      password: "",
      tok: localStorage.getItem("token"),
      isAuthorized: localStorage.getItem('isAuthorized'),
      greska_prijava: '',
      new_user_username: '',
      new_user_password: '',
      nova_sifra:'',
      provera_sifre:'',
      obavestenje_o_promeni:'',
      otkazani_drugar:null,
      isLoading: false
    }
  },

  created(){
    this.get_users()
    this.get_players()
    this.get_otkazane()
    this.get_drugare()  
  },

  methods:{
    get_users: async function () {
      const token = {
        headers: {'Authorization':'JWT '+ this.tok}
      }
      const res = await fetch("https://flask-football-app.herokuapp.com/users", token);
      const data = await res.json();

      this.users = data.users;
      if(data['status_code'] == 401){
        localStorage.setItem('isAuthorized', 'false')
        localStorage.removeItem('username')
      }
    },

    get_players: async function () {
      const gResponse = await fetch("https://flask-football-app.herokuapp.com/players");
      const gObject = await gResponse.json();
      this.players = gObject.players; 
    },

    get_otkazane: async function () {
      const gResponse = await fetch("https://flask-football-app.herokuapp.com/otkazani");
      const gObject = await gResponse.json();
      this.otkazani = gObject.otkazani
    },

    get_drugare: async function () {
      const res = await fetch("https://flask-football-app.herokuapp.com/drugari");
      const data = await res.json();
      this.drugari = data.drugari
    },

    promeniDrugara(drugar){
      this.otkazani_drugar = drugar.drugar
    },

    otkaziDrugara(){
      const requestOptions = {
        method: "DELETE",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/player/"+this.otkazani_drugar, requestOptions)
      const requestOptions1 = {
        method: "DELETE",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/drugar/"+this.otkazani_drugar, requestOptions1).then(this.reRenderAll())
    },

    otkazivanje () {
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/otkazan/"+this.username_show, requestOptions)
      const requestOptions1 = {
        method: "DELETE",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/player/"+this.username_show, requestOptions1).then(this.reRenderAll())
    },

    ponovnaPrijava(){
      const requestOptions = {
        method: "DELETE",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/otkazan/"+this.username_show, requestOptions).then(this.reRenderAll())
    },

    add_user(){
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username:this.new_user_username, password: this.new_user_password }),
      };
      fetch("https://flask-football-app.herokuapp.com/register", requestOptions);
      this.get_users()
    },

    autorizacija: function(){
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username:this.username, password:this.password}),
      };
      fetch("https://flask-football-app.herokuapp.com/auth", requestOptions)
      .then(res => res.json())
      .then(data => {
        if(data['access_token']){
        localStorage.setItem('isAuthorized', 'true')
        localStorage.setItem('token', data.access_token)
        localStorage.setItem('username', this.username) 
        this.tok = data.access_token
        window.location.reload()     
      }else{
        this.greska_prijava = data['description']
        console.log(this.greska_prijava)}
      })    
    },

    logout(){
      localStorage.removeItem('token')
      localStorage.removeItem('username')
      localStorage.setItem('isAuthorized', 'false')
      window.location.reload()
    },

    promenaSifre(){
      if(this.nova_sifra == this.provera_sifre){
        const requestOptions = {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ password:this.nova_sifra}),
      };
      fetch("https://flask-football-app.herokuapp.com/users/" + this.username_show, requestOptions)
      .then(res => res.json())
      .then(data =>{this.obavestenje_o_promeni = data.msg})
      }
      this.nova_sifra = ''
      this.provera_sifre = ''
    },

    reRenderAll(){
      this.$emit('change-component')
    }
  },
  computed:{

  }
}
</script>

<style scoped>
#pozdrav{
  /* margin: 10px; */
  font-size: 18px;
  font-weight: bold;
  text-align: center;
  border-radius: 5px;
  padding: 5px 20px;
  background-color: white;
  color: #126839;
}
.loading{
width: 100%;
height: 500px;
background-color: #268a53;
}
.login-form{
  padding: 0 20px;

}
.admin-form{
  background-color: #77a88c;
  border-radius: 10px;
  padding:15px 20px
}
label{
  margin-right: 20px;
}
.promeni-sifru input{
  margin-bottom: 5px;
}

.slicica-profil{
  width: 70px;
  height: 70px;
  background-size: cover;
  float: left;
  border-radius: 5px;
}
.login-form input[type=submit],
.login-form input[type=button]
{
  padding: 5px 15px;
  color: white;
  background-color: #268a53;
  border: none;
  border-radius: 5px;

}
.login-form input[type=text],
.login-form input[type=password]{
  padding-left: 10px;
  border-radius: 5px;
  border: none;
}
/* .login-form input:focus{
 background-color:  rgb(255, 245, 211);
} */
.otkazi input[type=submit] {
  background:  rgb(226, 39, 39);
  color: white;
  margin-bottom: 10px;
}
.prijavi-ponovo input[type=submit]{
  background:  rgb(29, 79, 228);
  color: white;
  margin-bottom: 10px;
}
.admin-panel{
  max-width: 700px;
  border: 1px solid gray;
  border-radius: 10px;
  padding: 20px 10px;
  margin-right: 15px;
  background-color: #99d4b3;
}

.admin-panel ul{
  list-style-type: none;
}
.admin-panel li{
  height: 30px;
  display: inline-block;
  margin: 3px 10px;
  background-color: #eafdf2;
  font-weight: bold;
  border-radius: 10px;
  min-width: 220px;
  max-width: 350px;
  padding-left: 2px;
  border-bottom: 2px solid green;
}

.status{
  padding: 0px 12px;
  background-color: rgb(94, 128, 228);
  /* border-top-left-radius: 8px;
  border-bottom-left-radius: 8px; */
  border-radius: 100%;
  margin-right: 10px;
  border-bottom: 2px solid rgb(102, 102, 102);
}
.slicica{
  width: 34px;
  height: 34px;
  background-size: cover;
  float: right;
  /* margin-top: -6px; */
 border-radius: 5px;
  margin-top: -2px;

}
.otkazao{
  background-color: rgb(226, 87, 87);
}
.prijavljen{
  background-color: rgb(57, 212, 96);
}
.obavestenje-prijava{
  font-weight: bold;
  color: #126839;
}
.obavestenje-otkazivanje{
  font-weight: bold;
  color: #681212;
}

.brankog{
  background-image: url('../assets/brankog.jpg');
}
.filiph{
  background-image: url('../assets/filiph.jpg');
}
.radosavb{
  background-image: url('../assets/radosavb.jpg');
}
.srdjanv{
  background-image: url('../assets/srdjanv.jpg');
}
.milosd{
background-image: url('../assets/milosd.jpg');
}
.vladimirb{
background-image: url('../assets/vladimirb.jpg');
}
.goranp{
  background-image: url('../assets/goranp.jpg');
}
.nikolao{
background-image: url('../assets/nikolao.jpg');
}
.nikolaz{
background-image: url('../assets/nikolaz.jpg');
}
.nikolac{
background-image: url('../assets/nikolac.jpg');
}
.markov{
background-image: url('../assets/markov.jpg');
}
.branislavm{
  background-image: url('../assets/branislavm.jpg');
}
.bogdanm{
    background-image: url('../assets/bogdanm.jpg');
}
.default{
    background-image: url('../assets/default.jpg');
}
.radovan{
    background-image: url('../assets/radovan.jpg');
}
.rastko{
    background-image: url('../assets/rastko.jpg');
}


</style>
