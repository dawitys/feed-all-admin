<template>
  <v-app id="app">
    <router-view app></router-view>
  </v-app>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
#app{
  background-color: #f0f0f0;
}
.extra-padding{
  padding: 25px;
}

</style>
