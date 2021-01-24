<template>
    <div id="Intro">
        <v-container fluid>
            <v-layout row justify-center>
            <v-flex md6 xs12>
                <v-card>
                    <v-card-title class="headline brown darken-2 white--text">Introduction</v-card-title>
                    <v-card-text>
                    <h3>{{intro.title}}</h3>
                    <v-spacer></v-spacer>
                    <p>{{intro.body}}</p>
                    </v-card-text>
                </v-card>
            </v-flex>
        </v-layout>
        </v-container>
        <v-btn color="brown darken-2" dark fixed bottom right fab @click="startEdit">
            <v-icon>fa-pencil</v-icon>
        </v-btn>

<!--EDIT INTRO MODAL-->
            <v-layout row justify-center>
            <v-dialog  v-model="editIntroDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span class="headline">Editing Introduction</span>
                </v-card-title>
                <v-card-text>
                    <v-container grid-list-md>
                        <v-layout wrap>
                            <v-flex xs12>
                                <v-form v-model="formValid" ref="form" lazy-validation>
                                    <v-text-field name="title" label="title" v-model="editingIntro.title" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field textarea name="body" label="body" v-model="editingIntro.body" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                </v-form>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="cancelEdit">Cancel</v-btn>
                <v-btn color="teal darken-1" flat @click.native="updateIntro" :disabled="!formValid">Submit</v-btn>
                </v-card-actions>
            </v-card>
            </v-dialog>
        </v-layout>
    </div>
</template>

<script>
import firebase from './firebaseInit'
import 'firebase/firestore'
const db = firebase.firestore()
export default {
  name: 'Intro',
  data () {
    return {
        formValid:true,
        intro: {
            id:null,
            title:null,
            body:null
        },
        editingIntro:{
            id:null,
            title:null,
            body:null
        },
        editIntroDialog:false
    }
  },
  methods:{
    getIntro(){
        db.collection('intro').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'title':doc.data().title,
                        'body':doc.data().body,
                    }
                    this.intro = data
                })
        })
    },
    startEdit(){
        this.editingIntro.id = this.intro.id
        this.editingIntro.title = this.intro.title
        this.editingIntro.body = this.intro.body
        this.editIntroDialog=true
    },
    cancelEdit(){
        this.editIntroDialog=false
        this.$refs.form.reset()
    },
    updateIntro(){
        
        const docRef = db.collection('intro').doc(this.editingIntro.id)
        docRef.set({
            title: this.editingIntro.title,
            body: this.editingIntro.body,
            })
        .then( (doc)=> {
            this.getIntro()
            console.log("Document successfully written!", doc);
            this.cancelEdit()
        })
        .catch(()=> {
            console.error("Error writing document: ", error)
            this.cancelEdit()
        })
    }
  },
  created(){
      this.getIntro()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
