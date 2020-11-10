<template>
  <div id="app" >
    <Navbar />
    <div id="wrapper">
      <div id="field"><Field :key="componentKey" @change-component="changeComponentKey()" msg="Welcome to Your Vue.js App"/> </div>
      <div id="admin-panel"><AdminPanel :key="componentKey1" @change-component="changeComponentKey()"/></div>
      <div id="chat"><Chat/></div> 
    </div>
    <Footer/>
</div>
</template>

<script>
import Field from './components/Field.vue'
import Navbar from './components/Navbar.vue'
import Chat from './components/Chat.vue'
import AdminPanel from './components/AdminPanel.vue'
import Footer from './components/Footer.vue'


export default {
  name: 'App',
  components: {
    Field,
    Navbar,
    Chat,
    AdminPanel,
    Footer
  },
  data(){
    return{
      title: "Fudbal",
      componentKey: 0,
      componentKey1: -100,
      
    }
  },
  methods:{
    changeComponentKey(){
      this.componentKey += 1;
      this.componentKey1 += 1;
    },
  }
 
}
</script>

<style>
#app{
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  background-color: #bcf0d3;


}
#wrapper{
  display: grid;
  grid-template-columns: 1.4fr 1fr;
  grid-gap:10px;
}
#wrapper > div{
  /* border: 1px solid red; */
}
/* #navbar{
grid-area: header;
}
#field{
grid-area: header;
}
#admin-panel{
grid-area: admin;
}
#chat{
grid-area: chat;
} */

</style>
