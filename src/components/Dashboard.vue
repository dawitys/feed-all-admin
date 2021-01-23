<template>
<div id="Dashboard">
  <v-container fluid class="bg">
    <div class="flex-wrapper white--text">
        <h2 class="display-3" >Welcome to Admin Panel</h2>
        <v-btn id="linkToLota" color="pink darken-2" round large dark href="https://www.apartmani-lota.com" target="_blank">Lota Home <v-icon right>fa-external-link</v-icon></v-btn>
    </div>
  </v-container>
</div>
</template>

<script>
export default {
  name: 'Dashboard',
  data () {
    return {
      
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#Dashboard, .container, .layout, .flex{
  height:100%;
}
/*.bg{
  background-image: url('../assets/img/dashboard.jpg');
  background-color: rgba(10, 99, 80, 0.685);
  background-size:cover;
  background-blend-mode:multiply ;
}*/
.flex-wrapper{
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  justify-content: center;
  height: 100%;
  padding: 50px;
}
#linkToLota{
  margin-right: 0;
}
</style>
