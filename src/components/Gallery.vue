<template>
  <div id="Gallery" >
      <!--GALLERY TABLE-->
        <v-container fluid>
            <v-layout row justify-center>
            <v-flex md11 xs12>
                <v-card>
                    <v-card-title class="headline pink darken-2 white--text">Gallery</v-card-title>
                    <v-data-table
                        v-bind:headers="tableHeaders"
                        :items="homePhotos"
                        hide-actions
                        class="elevation-1"
                        >
                        <template slot="items" slot-scope="props">
                            <td><img :src="props.item.url" class="preview pointer" @click="openPreview()"></td>
                            <td class="text-xs-right">{{ props.item.position }}</td>
                            <td class="text-xs-right">{{ props.item.name }}</td>
                            <td class="text-xs-right">{{ props.item.path }}</td>
                            <td class="text-xs-right">
                                <v-btn icon  @click="startPosition(props.item)"><v-icon>fa-pencil</v-icon></v-btn>
                                <v-btn icon  @click="deletePhoto(props.item, 'homePhotos')"><v-icon>fa-times</v-icon></v-btn>
                            </td>
                        </template>
                    </v-data-table>
                </v-card>
            </v-flex>
        </v-layout>   
        </v-container> 
        <v-btn color="pink darken-2" dark fixed bottom right fab @click.native.stop="uploadDialog = !uploadDialog">
            <v-icon>fa-upload</v-icon>
        </v-btn>
<!--PREVIEW MODAL-->
<v-layout row>
    <v-dialog v-model="previewDialog" max-width="768px">
      <v-card class="text-xs-center">
            <v-carousel hide-delimiters>
                <v-carousel-item class="fit" v-for="(item,i) in homePhotos" v-bind:src="item.url" :key="i"></v-carousel-item>
            </v-carousel>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" flat="flat" @click.native="closePreview()">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
<!--UPLOAD MODAL-->
        <v-layout row justify-center>
            <v-dialog v-model="uploadDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span class="headline">Photo Upload</span>
                </v-card-title>
                <v-card-text>
                <v-container grid-list-md>
                    <v-layout wrap>
                    <form>
                        <input type="file" multiple accept="image/*" @change="processFiles($event)" required>  
                         <!--<v-progress-linear v-model="uploadProgress"></v-progress-linear>-->
                    </form>
                    </v-layout>
                </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="cancelUpload()">Close</v-btn>
                <v-btn color="teal darken-1" flat @click.native="uploadPhotos()">Upload</v-btn>
                </v-card-actions>
            </v-card>
            </v-dialog>
        </v-layout>

        <!--POSITION PHOTO MODAL-->
        <v-layout row justify-center>
            <v-dialog v-model="positionDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span class="headline">Position Photo</span>
                </v-card-title>
                <v-card-text>
                <v-container grid-list-md>
                    <v-layout wrap>
                    <v-flex xs12>
                        <v-form v-model="valid">
                        <v-text-field
                        name="position"
                        label="Position"
                        id="position"
                        type="number"
                        v-model="currentPhoto.position"
                        required
                        ></v-text-field>
                    </v-form>
                    </v-flex>
                    </v-layout>
                </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="cancelPosition()">Cancel</v-btn>
                <v-btn color="teal darken-1" flat @click.native="positionPhoto(currentPhoto, 'homePhotos')" :disabled="!valid">Submit</v-btn>
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
const imagesRef = storageRef.child('home')

export default {
  name: 'Gallery',
  data () {
    return {
        valid: true,
        photosToUpload: [],
        homePhotos: [],
        postPhotos:[],
        tableHeaders:[{text:"preview", value: "url"},{text:"Position", value: "position"},{text:"name", value: "name"},{text:"path", value: "path"},{text:"options", value: "options",sortable: false,}],
        selectedPath: "",
        folders:[{text:"Gallery", value: "homePhotos"}, {text: "Reviews", value:"postPhotos"}],
        uploadProgress:null,
        uploadOn:false,
        previewOn: false,
        photoPreview: null,
        uploadDialog: false,
        previewDialog: false,
        positionDialog:false,
        currentPhoto: {
            'id': null,
            'name': null,
            'path':null,
            'url':null,
            'position':null,
        }
    }
  },
  methods:{
    listPhotos(){
        this.homePhotos=[]
        this.postPhotos=[]
        db.collection("homePhotos").orderBy('position').get().then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        const homePhoto = {
                            'id': doc.id,
                            'name': doc.data().name,
                            'path':doc.data().path,
                            'url':doc.data().url,
                            'position':doc.data().position
                        }
                        this.homePhotos.push(homePhoto)
                        //console.log("photos", homePhoto.position)
                    })
        })
    },
    processFiles(event) {
        this.photosToUpload = event.target.files
    },
    uploadPhotos(){
        this.uploadOn=true;
        const ref = 'homePhotos'
        Array.from(this.photosToUpload).forEach(photo => { 
            var uploadTask = storageRef.child(ref +"/"+ photo.name).put(photo);
            // Register three observers:
            // 1. 'state_changed' observer, called any time the state changes
            // 2. Error observer, called on failure
            // 3. Completion observer, called on successful completion
            uploadTask.on('state_changed', function(snapshot){
            // Observe state change events such as progress, pause, and resume
            // Get task progress, including the number of bytes uploaded and the total number of bytes to be uploaded
            this.uploadProgress = parseInt((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
            console.log('Upload is ' + this.uploadProgress + '% done');
            
            }, error => {
            // Handle unsuccessful uploads
            }, () => {
            // Handle successful uploads on complete
            // For instance, get the download URL: https://firebasestorage.googleapis.com/...
            console.log("snapshot", uploadTask.snapshot)
            var downloadURL = uploadTask.snapshot.downloadURL;
            // Add a new document with a generated id.
            var photoObj ={
                url: downloadURL,
                path: ref + "/" + photo.name,
                name: photo.name
            }
            db.collection(ref).add(photoObj)
                    .then(docRef=> {
                        this.listPhotos()
                        console.log("Document written with ID: ", docRef.id);
                    })
                    .catch(error=> {
                        console.error("Error adding document: ", error);
                })
            })
            this.uploadOn= false
            this.uploadDialog= false
            this.photosToUpload= []
         })
    },
    cancelUpload(){
        this.uploadDialog= false
        this.photosToUpload = []
    },
    positionPhoto(photo, collection){
        const docRef = db.collection(collection).doc(photo.id)
      docRef.set({
          id: photo.id,
          name: photo.name,
          path: photo.path,
          url: photo.url,
          position: photo.position
        })
        .then( (doc)=> {
            this.cancelPosition()
        })
        .catch(()=> {
            console.error("Error editing document: ", error);
      })
    },
    cancelPosition(){
      this.positionDialog = false
      this.currentPhoto = {
        'id': null,
        'name': null,
        'path':null,
        'url':null,
        'position':null,
      }
    },
    startPosition(photo){
      this.positionDialog=true
      this.currentPhoto = photo
    },
    deletePhoto(photo, collection){
        if(confirm('Are you sure?')){
            var desertRef = storageRef.child(photo.path);
            // Delete the file
            desertRef.delete().then(function() {
            // File deleted successfully
            }).catch(function(error) {
            // Uh-oh, an error occurred!
            })
            db.collection(collection).doc(photo.id).delete()
                .then(data => {
                    this.listPhotos()
                    console.log("Document successfully deleted!");
                }).catch(error => {
                    console.error("Error removing document: ", error);
            })
        }
        
    },
    openPreview(){
        this.previewDialog=true
    },
    closePreview(){
        this.previewDialog=false
    }
    },
    created(){
        this.listPhotos()
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#Gallery{
    height: 100%;
}
.fit{
    max-width:100%;
}
.preview{
    width: 300px;
    max-width: 100vw;
}
@media only screen and (max-width: 900px){
    .preview{
    width: 100px;
    max-width: 100vw;
}
}
.pointer{
    cursor:pointer;
}
</style>
