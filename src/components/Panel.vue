<template>
  <div id="Panel">
      <v-app>
        <v-toolbar app color="#083110" dark>
          <v-toolbar-side-icon @click="toggleNav"><v-icon>fa-bars</v-icon></v-toolbar-side-icon>
          <v-toolbar-title>FeedAll Admin</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click="firebaseLogout">
            <v-icon icon class="right">fa-sign-out</v-icon>
          </v-btn>
        </v-toolbar>
        <v-content app class="bg">
          <router-view></router-view>
        </v-content>
        <v-navigation-drawer app light fixed v-model="navActive">
          <div class="black-logo">&nbsp;</div>
          <v-divider></v-divider>
          <v-list dense class="pt-0">
            <v-list-tile class="list__tile--link" v-for="item in nav" :key="item.title" @click="goTo(item.route)">
              <v-list-tile-action>
                <v-icon>{{ item.icon }}</v-icon>
              </v-list-tile-action>
              <v-list-tile-content>
                <v-list-tile-title>{{ item.title }}</v-list-tile-title>
              </v-list-tile-content>
            </v-list-tile>
          </v-list>
        </v-navigation-drawer>
      <v-footer app color="teal"></v-footer>
    </v-app>
  </div>
</template>

<script>
import router from '../router'
import firebase from './firebaseInit'
export default {
  name: 'Panel',
  data () {
    return {
      navActive: true,
      nav:[
        {title: "Dashboard", route:"/dashboard", icon:"fa-bar-chart"},
        {title: "Introduction", route:"/intro", icon:"fa-comment-o"},
        {title: "Intro Background", route:"/introbg", icon:"fa-file-image-o"},
        {title: "Services", route:"/services", icon:"fa-bell-o"},
        {title: "Gallery", route:"/gallery", icon:"fa-picture-o"},
        {title: "Reviews", route:"/reviews", icon:"fa-star-o"}
      ]
    }
  },
  methods:{
     firebaseLogout(){
        firebase.auth().signOut().then(()=> {
        // Sign-out successful.
        }).catch(function(error) {
        // An error happened.
        });
      },
      toggleNav(){
        this.navActive = !this.navActive;
      },
      goTo(route){
        router.push(route)
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#Panel{
    height:100%;
}
div .list__tile__action{
  width:auto;
}
.bg{
  background-image: url('../assets/img/dashboard.jpg');
  background-color: rgba(140, 227, 111, 0.685);
  background-size:cover;
  background-blend-mode:multiply ;
  background-repeat:no-repeat;
  background-attachment: fixed;
}
.black-logo{
  height: 21rem;
  width: 21rem;
  content:url('../assets/img/logo_web.png');
  //background-image: url('../assets/img/logo_web.png');
  }

</style>
