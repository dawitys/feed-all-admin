<template>
  <div id="Login">
    <v-app>
        <v-container fluid fill-height>
        <v-layout flex align-center justify-center>
          <v-flex md3 xs11>
               <form @submit.prevent="firebaseLogin">
                <v-card>
                    <v-toolbar class="teal">
                        <div class="stretch headline white--text text-xs-center">FeedAll Admin</div>
                    </v-toolbar>
                    <v-card-text>
                        <v-layout column>
                            <v-flex xs10 offset-xs1>
                                <v-text-field v-model="email" color="teal" name="email" type="text" label="Email" id="email" required></v-text-field>
                            </v-flex>
                            <v-flex xs10 offset-xs1>
                                <v-text-field v-model="password" color="teal" name="password" type="password" label="Password" id="password" required></v-text-field>
                            </v-flex>
                        </v-layout>
                    </v-card-text>
                    <v-card-actions class="justify-center">
                        <v-btn dark color="teal" type="submit">LOGIN</v-btn>
                    </v-card-actions>
                </v-card>
                </form>
            </v-flex>
      </v-layout>
    </v-container>
    </v-app>
  </div>
</template>

<script>
import firebase from './firebaseInit'
import router from '../router'
export default {
  name: "Login",
  data() {
    return {
      email: null,
      password: null
    };
  },
  methods:{
      
      

      firebaseLogin(){
        firebase.auth().signInWithEmailAndPassword(this.email, this.password)
        .then((user)=>{
            
            router.push('/panel')
        })
        .catch((error)=> {
                // Handle Errors here.
                var errorCode = error.code
                var errorMessage = error.message
                // ...
                console.log("error code:", errorCode, "error message", errorMessage)
        });
      }
      
  }
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

#Login{
      height: 100vh;
}
.toolbar{
    padding: 15px 0;
}
.card__text{
    padding: 35px 25px;
}
.input-group{
    padding-top:25px;
}
.card__actions{
    border-top: 1px solid rgba(0,0,0,0.1);
    padding: 15px 4px;
}
.btn{
    padding: 0 15px;
}
.stretch{
    width:100%;
}
</style>
