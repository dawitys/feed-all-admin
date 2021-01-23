<template>
    <div id="Reviews">
        <!--Review TABLE-->
       <v-container fluid>
            <v-layout row justify-center>
            <v-flex md11 xs12>
                <v-card>
                    <v-card-title class="headline pink darken-2 white--text">Reviews</v-card-title>
                    <v-data-table v-bind:headers="tableHeaders" :items="reviews" hide-actions class="elevation-1">
                        <template slot="items" slot-scope="props">
                            <td class="text-xs-right"><img :src="props.item.profileUrl" class="profile-preview"></td>
                            <td class="text-xs-right">{{ props.item.position }}</td>
                            <td class="text-xs-right">{{ props.item.name }}</td>
                            <td class="text-xs-right">{{ props.item.comment }}</td>
                            <td class="text-xs-right">
                                <v-btn icon @click="startEdit(props.item)"><v-icon>fa-pencil</v-icon></v-btn>
                                <v-btn icon  @click="deleteReview(props.item)"><v-icon>fa-times</v-icon></v-btn>
                            </td>
                        </template>
                    </v-data-table>
                </v-card>
            </v-flex>
        </v-layout>
       </v-container>
        <v-btn color="pink darken-2" dark fixed bottom right fab @click.native.stop="newReviewDialog = !newReviewDialog">
            <v-icon>fa-plus</v-icon>
        </v-btn>
        <!--NEW SERVICE MODAL-->
            <v-layout row justify-center>
            <v-dialog  v-model="newReviewDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span v-if="!editOn" class="headline">Create New Review</span>
                <span v-else class="headline">Edit Review</span>
                </v-card-title>
                <v-card-text>
                    <v-container grid-list-md>
                        <v-layout wrap>
                            <v-flex xs12>
                                <v-form v-model="valid" ref="reviewForm" lazy-validation>
                                    <v-text-field name="name" label="name" v-model="newReview.name" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field textarea name="comment" label="comment" v-model="newReview.comment" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field type="number" name="position" label="position" v-model="newReview.position" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <input id="photoUploadInput" name="photo" type="file" accept="image/*" @change="processFiles($event)" required> 
                                </v-form>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="resetNewReview()">Cancel</v-btn>
                <v-btn v-if="!editOn" color="teal darken-1" flat @click.native="createNewReview()" :disabled="!valid">Submit</v-btn>
                <v-btn v-else color="teal darken-1" flat @click.native="updateReview()" :disabled="!valid">Submit</v-btn>
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
  name: 'Reviews',
  data () {
    return {
        newReviewDialog: false,
        valid: true,
        newReview: {
            id:null,
            profileUrl: null,
            name: null,
            comment: null,
            position:null,
            profilePhoto:null
        },
        tableHeaders: [
            {text:"Icon", value: "profileUrl"},
            {text:"Position", value: "position"},
            {text:"Title", value: "name"},
            {text:"Description", value: "comment"}
        ],
        reviews: [],
        reviewsLoading:false,
        editOn:false
    }
  },
  methods:{
    listReviews(){
        this.reviewsLoading = true
        this.reviews=[]
        db.collection('reviews').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'profileUrl': doc.data().profileUrl,
                        'profilePath': doc.data().profilePath,
                        'name':doc.data().name,
                        'comment':doc.data().comment,
                        'position':doc.data().position
                    }
                    this.reviews.push(data)
                })
        })
        this.reviewsLoading = false
    },
    resetNewReview(){
        this.newReview= {
            id:null,
            profileUrl: null,
            name: null,
            comment: null,
            position:null,
            profilePhoto:null
        }
        this.editOn=false
        this.newReviewDialog=false
        this.$refs.reviewForm.reset()
        var field = document.getElementById('photoUploadInput');
        field.value= field.defaultValue;
    },
    startEdit(review){
        this.newReview= {
            id:review.id,
            profileUrl: review.profileUrl,
            profilePath: review.profilePath,
            name: review.name,
            comment: review.comment,
            position: review.position
        }
        this.editOn = true
        this.newReviewDialog =true
    },
    updateReview(){
        if( !this.newReview.name|| !this.newReview.comment || !this.newReview.position)
            return
        if(this.newReview.profilePhoto){
            this.deletePhoto(this.newReview.profilePath)
            Array.from(this.newReview.profilePhoto).forEach(photo => { 
              const ref = 'profilePhotos'
              var uploadTask = storageRef.child(ref +"/"+ this.newReview.name).put(photo)
              uploadTask.on('state_changed',function(snapshot){
                var uploadProgress = parseInt((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
                console.log('Upload is ' + uploadProgress + '% done');
                }, error => {
                  console.log(error)
                }, () => {
                  var downloadURL = uploadTask.snapshot.downloadURL;
                  var filePath = uploadTask.snapshot.metadata.fullPath;
                  const docRef = db.collection('reviews').doc(this.newReview.id)
                  docRef.set({
                      name: this.newReview.name,
                      comment: this.newReview.comment,
                      position: this.newReview.position,
                      profileUrl: downloadURL,
                      profilePath: filePath
                  })
                  .then( (doc)=> {
                      this.listReviews()
                      console.log("Document successfully written!", doc)
                      this.resetNewReview()
                  })
                  .catch(()=> {
                  console.error("Error writing document: ", error)
                  })
              })
          })
        }else{
          const docRef = db.collection('reviews').doc(this.newReview.id)
            docRef.set({
                name: this.newReview.name,
                comment: this.newReview.comment,
                position: this.newReview.position,
                profileUrl: this.newReview.profileUrl,
                profilePath: this.newReview.profilePath
            })
            .then( (doc)=> {
                this.listReviews()
                console.log("Document successfully written!", doc)
                this.resetNewReview()
            })
            .catch(()=> {
            console.error("Error writing document: ", error)
            })
        }
        
       
    },
    deletePhoto(path){
          var desertRef = storageRef.child(path);
          desertRef.delete().then(function() {
          console.log("image deleted");
          }).catch(function(error) {
          console.log("error", error)
          })
    },
    deleteReview(review){
        if(confirm("Are you sure?")){
        this.deletePhoto(review.profilePath)
        db.collection('reviews').doc(review.id).delete()
        .then(data => {
            this.listReviews()
            console.log("Document successfully deleted!");
        }).catch(error => {
            console.error("Error removing document: ", error);
        })
      }
    },
    processFiles(event) {
        this.newReview.profilePhoto = event.target.files
    },
    createNewReview(){
        const ref = 'profilePhotos'
        Array.from(this.newReview.profilePhoto).forEach(photo => { 
          console.log("review",this.newReview)
          var uploadTask = storageRef.child(ref +"/"+ this.newReview.name).put(photo)
          uploadTask.on('state_changed', function(snapshot){
          var uploadProgress = parseInt((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
          console.log('Upload is ' + uploadProgress + '% done');
          }, error => {
            console.log(error)
          }, () => {
            
          var downloadURL = uploadTask.snapshot.downloadURL;
          var filePath = uploadTask.snapshot.metadata.fullPath;
          db.collection('reviews').add({
                  profileUrl: downloadURL,
                  profilePath: filePath,
                  name: this.newReview.name,
                  comment: this.newReview.comment,
                  position: this.newReview.position
              })
          .then(docRef => {
                  this.listReviews()
                  this.resetNewReview()
                  this.newReviewDialog = false
              })
          .catch(error => console.log(err))
          })
      })
    }
  },
  created(){
      this.listReviews()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.profile-preview{
  width: 120px;
  height: 120px;
  object-fit:cover;
}

</style>
