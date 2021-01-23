<template>
    <div id="IntroBg">
        <v-container fluid>
            <v-layout row justify-center>
            <v-flex md6 xs12>
                <v-card>
                    <v-card-title class="headline pink darken-2 white--text">Picture</v-card-title>
                    <v-card-text class="text-xs-center">
                        <img :src="introBg.url" class="fit">
                    </v-card-text>
                </v-card>
            </v-flex>
        </v-layout>
        </v-container>
        <v-btn color="pink darken-2" dark fixed bottom right fab @click.native.stop="uploadDialog = !uploadDialog">
            <v-icon>fa-upload</v-icon>
        </v-btn>

        <v-layout row justify-center>
            <v-dialog v-model="uploadDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span class="headline">Photo Upload</span>
                </v-card-title>
                <v-card-text>
                <v-container grid-list-md>
                    <v-layout wrap>
                    <v-form ref="form" v-model="formValid">
                        <input id="photoUploadInput" type="file" accept="image/*" @change="processFiles($event)" required>  
                         <!--<v-progress-linear v-model="uploadProgress"></v-progress-linear>-->
                    </v-form>
                    </v-layout>
                </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="cancelUpload()">Close</v-btn>
                <v-btn color="teal darken-1" flat @click.native="uploadPhotos()" :disabled="!newPhoto">Upload</v-btn>
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
const storageRef = firebase.storage().ref()

export default {
  name: 'IntroBg',
  data () {
    return {
        formValid:true,
        uploadDialog:false,
        introBg: {
            id:null,
            url:null,
            path:null
        },
        newPhoto: null
    }
  },
  methods:{
    getIntroBg(){
        db.collection('introBg').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'url':doc.data().url,
                        'path':doc.data().path
                    }
                    this.introBg = data
                })
        })
    },
    uploadPhotos(){
        const ref = 'introBg'
        const photoId = this.introBg.id
        if(this.introBg.path) this.deletePhoto(this.introBg.path)
        Array.from(this.newPhoto).forEach(photo => { 
            var uploadTask = storageRef.child(ref +"/"+ photoId).put(photo);
            uploadTask.on('state_changed', function(snapshot){
            //this.uploadProgress = parseInt((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
            //console.log('Upload is ' + this.uploadProgress + '% done');
            }, error => {
                console.log("upload unsuccessful")
            }, () => {
            var downloadURL = uploadTask.snapshot.downloadURL;
            var filePath = uploadTask.snapshot.metadata.fullPath;
            const docRef = db.collection(ref).doc(photoId)
                  docRef.set({
                      url: downloadURL,
                      path: filePath
                  })
                  .then( (doc)=> {
                      console.log("Document successfully written!", doc)
                      this.getIntroBg()
                  })
                  .catch(()=> {
                  console.error("Error writing document: ", error)
                  })
            })
        })
    },
    cancelUpload(){
        var field = document.getElementById('photoUploadInput')
        field.value= field.defaultValue
        this.uploadDialog= false
        this.newPhoto = null
        this.$refs.form.reset()
    },
    processFiles(event) {
        this.newPhoto = event.target.files
    },
    deletePhoto(path){
        var desertRef = storageRef.child(path);
        desertRef.delete().then(function() {
        //console.log("image deleted");
        }).catch(function(error) {
        console.log("error", error)
        })
    }
  },
  created(){
      this.getIntroBg()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.fit{
    max-width:100%;
}

</style>
