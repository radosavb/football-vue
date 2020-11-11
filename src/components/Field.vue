<template >
  <div >
    <!-- Uputstva za prijavu! -->
    <div v-show="username" id="uputstva">
        > Za potvrdu dolaska se prijavi klikom na slobodnu poziciju (fudbalsku loptu) na terenu. Takodje mozes da prijavis gej drugara tako sto ces uneti njegovo ime i kliknuti na dugme "Prijavi drugara".<br>
        > Za otkazivanje dolaska klikni na dugme "Otkazi igru" ili otkazi za drugara klikom na odgovarajuce crveno dugme.<br>
        > Ako si vec otkazao, ponovo se vrati u "neizjasnjene" klikom na plavo dugme.<br>
        <b class="text-danger">>>>Ako promena nije odmah prikzana ponovo učitaj stranicu!!!</b>
    </div>
    <!-- Broj prijavljenih -->
  <div class="field">  
    <span v-if="players.length < 10" class="prijavljeni bolder rounded  h5 bg-light p-1">Trenutno prijavljeno <b class="bg-dark text-light p-1">{{players.length}}</b> igraca</span>
    <span v-else class="h4 text-light bg-danger p-3 rounded bold ml-3 "> URAAAAAA!!! IGRA SE!!! <b class="rounded-circle text-danger p-2 bg-light">{{players.length}}</b> GEJEVA SE PRIJAVILO!!!</span>
    <!-- Teren -->
    <div  v-for="(polje, index) in polja"  @click="[selectedIndex(index), modalf()]" class="player" :class="[{default_slika: pozicije.indexOf(polje.name) == -1 && stalni_igraci.indexOf(polje.name) == -1},{blur: pozicije.indexOf(polje.name) == -1}, polje.position, polje.name]"   :key="polje.position" >{{polje.name}} </div>
    <!-- Modal za prijavu -->
     <div v-show="modal" class="mod">
       <span @click="modal=false" class="float-right">x</span>
       <div class="slicica" :class="username" @click="[registruj(), ponovnaPrijava()]">Prijavi sebe</div>
    <hr>
      <form @submit.prevent="[registruj_drugara(), ponovnaPrijava()]">       
        <small>ili unesi ime drugara i klikni na dugme</small>               
        <input placeholder="" type="text" v-model="drugar" />
        <input type="submit" value="Prijavi drugara"/>
      </form>
    </div>
  </div>
  <div v-show="zauzeto" class=" zauzeto alert alert-danger alert-dismissible fade show" role="alert">
  <strong>Zauzeto!</strong> Izaberite drugu poziciju.
  <button @click="zauzeto=false" type="button" class="close" data-dismiss="alert" aria-label="Close">
    <span aria-hidden="true">&times;</span>
  </button>
</div>
<div v-show="obavestenje_reg_drugara" class=" alert alert-info alert-dismissible fade show" role="alert">
  <strong>!</strong> {{obavestenje_reg_drugara}}
  <button @click="obavestenje_reg_drugara=''" type="button" class="close" data-dismiss="alert" aria-label="Close">
    <span aria-hidden="true">&times;</span>
  </button>
</div>
  </div>
</template>

<script>

export default {
  name: 'Field',
  props: {
    msg: String
  },
  data(){
      return {
          polja: [
        { name: "gk_home", position: "gk_home" },
        { name: "dmf_home", position: "dmf_home" },
        { name: "aml_home", position: "aml_home" },
        { name: "amr_home", position: "amr_home" },
        { name: "cf_home", position: "cf_home" },
        { name: "sub1_home", position: "sub1_home" },
        { name: "sub2_home", position: "sub2_home" },
        { name: "gk_away", position: "gk_away" },
        { name: "dmf_away", position: "dmf_away" },
        { name: "aml_away", position: "aml_away" },
        { name: "amr_away", position: "amr_away" },
        { name: "cf_away", position: "cf_away" },
        { name: "sub1_away", position: "sub1_away" }, 
        { name: "sub2_away", position: "sub2_away" }, 

      ],
        pozicije: ["gk_home","dmf_home","aml_home","amr_home", "cf_home","sub1_home","sub2_home","gk_away","dmf_away","aml_away","amr_away","cf_away", "sub1_away", "sub2_away"],
        stalni_igraci:['radosavb', 'brankog', 'filiph', 'milosd', 'vladimirb', 'vladimirz', 'nikolaz', 'nikolao', 'nikolac', 'goranp', 'srdjanv', 'milosj', 'markov', 'branislavm', 'bogdanm'],
         players:[],
         users:[],
         player_index:null,
         username: localStorage.getItem('username'),
         tok: localStorage.getItem('token'),
         modal: false,
         drugar: "",
         moji_igraci:[],
         zauzeto: false,
         response: '',
         obavestenje_reg_drugara: '',
         kc: 0
      }
  },
  methods:{
    fUpdate(){
      this.forceUpdate()
    },
    selectedIndex(index){
      this.player_index = index
      // console.log(this.player_index)
    },
    proba(){
      console.log('object')
    },
    modalf(){
      if(this.pozicije.indexOf(this.polja[this.player_index].name) > -1){
        this.modal = true
        this.zauzeto = false
      }else{
        this.zauzeto = true
      }
    },
    ispis(){
      console.log(this.player_index)
    },
    registruj: async function () {
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json", "Authorization": "JWT "+this.tok },
        body: JSON.stringify({ position: this.polja[this.player_index].position }),
      };
      fetch("https://flask-football-app.herokuapp.com/player/"+this.username, requestOptions).then(this.get_players()).then(this.reRenderAll())
      this.modal = false
      
    },
    registruj_drugara: async function () {
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json", "Authorization": "JWT "+this.tok },
        body: JSON.stringify({ username: this.username, position: this.polja[this.player_index].position }),
      };
      fetch("https://flask-football-app.herokuapp.com/drugar/"+this.drugar, requestOptions)
      
      const requestOptions1 = {
        method: "POST",
        headers: { "Content-Type": "application/json", "Authorization": "JWT "+this.tok },
        body: JSON.stringify({ position: this.polja[this.player_index].position }),
      };
      fetch("https://flask-football-app.herokuapp.com/player/"+this.drugar, requestOptions1).then(this.get_players()).then(this.reRenderAll())
      this.modal = false

    },
    ponovnaPrijava(){
      const requestOptions = {
        method: "DELETE",
        headers: { "Content-Type": "application/json" },
      };
      fetch("https://flask-football-app.herokuapp.com/otkazan/"+this.username, requestOptions).then(this.get_players()).then(this.reRenderAll())
    },

    get_players: async function () {
      const res = await fetch("https://flask-football-app.herokuapp.com/players");
      const data = await res.json();
      this.players = data.players; 
      this.update_players()
    },
    get_users: async function () {
      const token = {
        headers: {'Authorization':'JWT '+ this.tok}
      }
      const res = await fetch("https://flask-football-app.herokuapp.com/users", token);
      const data = await res.json();

      this.users = data.users;
      if(data['status_code'] == 401){
        this.isAuthorized = 'false'
        localStorage.removeItem('username')
      }
    },
    update_players() {
      let brojac = 0;
      for (let i = 0; i < this.players.length; i++) {
        for (let j = 0; j < this.polja.length; j++) {
          if (this.players[brojac].position == this.polja[j].position) {
            this.polja.splice(j, 1, this.players[brojac]);
            brojac++;
          }
        }
      }
    },
    reRenderAll(){
      this.$emit('change-component')
    }
  }, 
  created(){  
    this.get_players()   
  },  

}
</script>

<style scoped>
#uputstva {
  padding: 5px;
  border-radius: 8px;
  margin-left: 15px;
  line-height: 15px;
  margin-bottom: 15px;
  font-weight: bold;
  border: 1px solid rgb(124, 124, 124);
}

.prijavljeni{
  border: 2px solid gray;
}
small{
  font-weight: 800;
}
.field {
  position: relative;
  width: 900px;
  height: 600px;
  background-image: url('../assets/field.jpg');
  background-repeat: no-repeat;
  background-size: 100%;
  margin: 0 auto
}
.slicica{
  background-size: contain;
}
.blur{
  box-shadow:0px -8px 15px 10px rgb(255, 253, 109)
}
.zauzeto{
  margin-top: -400px;
  width: 300px;
  margin-left: auto;
  margin-right: auto;
}
.mod{
  display: block;
  width: 300px;
  border-radius:25px;
  padding: 10px;
  border: 2px solid rgb(42, 52, 99);
  z-index:9999;
  background-color: rgb(157, 199, 255);
  text-align: center;
  position: absolute;
  top: 100px;
  left: 300px;
}
.mod input[type=submit]{
  margin: 5px auto;
  padding: 5px 15px;
  border-radius: 5px;
  background-color:rgb(62, 96, 207);
  color: white;
  font-weight: 700;
  border: none;
}
.mod input[type=text]{
border-radius: 5px;
border: none;
padding: 5px;
}

.mod > div{
  /* background-color: rgb(24, 24, 201); */
  width:110px;
  height: 110px;
  color:white;
  text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;;
  font-weight: 700;
  margin: auto;
  padding-top: 90px;
  border-radius: 100%;
  cursor: pointer;
}
.mod span{
  width: 35px;
  font-weight: bold;
  border: 1px solid rgb(153, 153, 153);
  border-radius: 50%;
  padding: 3px;
  text-align: center;
  color: white;
  background-color:  rgb(62, 96, 207);
  cursor: pointer;
  margin-right: -20px;
  margin-top: -20px;
}
.player {
  width: 80px;
  height: 80px;
  border-radius: 100%;
  border: 1px solid black;
  line-height: 160px;
  color: white;
  text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;
  text-align: center;
  cursor: pointer;
  position: absolute;
  background-size: cover;
  /* background-color: antiquewhite; */
  background-image: url(../assets/lopta1.jpg);
  font-weight: 700;
}
.gk_home {
  top: 250px;
  left: 3%;
  border: 2px solid royalblue;
}
.dmf_home {
  top: 250px;
  left: 20%;
  border: 2px solid royalblue;
}
.aml_home {
  top: 40px;
  left: 30%;
  border: 2px solid royalblue;
}
.amr_home {
  top: 460px;
  left: 30%;
  border: 2px solid royalblue;
}
.cf_home {
  top: 250px;
  left: 38%;
  border: 2px solid royalblue;
}
.sub1_home {
  top: 570px;
  left: 6%;
  border: 2px solid royalblue;
}
.sub2_home {
  top: 570px;
  left: 18%;
  border: 2px solid royalblue;
}
.gk_away {
  top: 250px;
  left: 88%;
  border: 2px solid rgb(209, 58, 209);
}
.dmf_away {
  top: 250px;
  left: 70%;
  border: 2px solid rgb(209, 58, 209);
}
.amr_away {
  top: 40px;
  left: 61%;
  border: 2px solid rgb(209, 58, 209);
}
.aml_away {
  top: 460px;
  left: 61%;
  border: 2px solid rgb(209, 58, 209);
}
.cf_away {
  top: 250px;
  left: 53%;
  border: 2px solid rgb(209, 58, 209);
}
.sub1_away {
  top: 570px;
  left: 72%;
  border: 2px solid rgb(209, 58, 209);
}
.sub2_away {
  top: 570px;
  left: 85%;
  border: 2px solid rgb(209, 58, 209);
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
.radovan{
    background-image: url('../assets/radovan.jpg');
}
.rastko{
    background-image: url('../assets/rastko.jpg');
}
.vladimirm{
    background-image: url('../assets/vladimirm.jpg');
}
.vukasink{
    background-image: url('../assets/vukasink.jpg');
}
.default_slika{
    background-image: url('../assets/default.jpg');
}
.ukloni-igraca{
  position: absolute;
  top: -75px;
  left: 65px;
  cursor: pointer;
  padding: 5px;
  /* z-index: 9999; */
}
</style>
