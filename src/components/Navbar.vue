<template>
  <nav>
      <div class="nav-bar">
        <span class="logo">Geh Football</span>
            
        <span class="termin">Sledeći termin:<b class="termin-datum"> {{termin}}</b></span>
      </div>
  </nav> 
</template>

<script>


export default {  
  data(){
    return{
      termin: "Nedelja 15.11.2020"
    }
  },
}
</script>

<style>
.nav-bar {
  background: linear-gradient(-90deg, #2b813e, #0c0c0c);
  height: 60px;
  margin-bottom: 15px;
  color: white;

}
.logo {
  margin-right: 30px;
  margin-left: 20px;
  font-weight: 700;
  font-size: 25px;
  line-height: 60px;
}
.termin {
  float: right;
  margin-right: 20px;
  line-height: 60px;
  
}
.termin-datum{

}
.dugmici {
  padding: 7px 12px;
  border: 1px solid white;
  font-size: 15px;
  margin-right: 10px;
  background-color: green;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
}
</style>
