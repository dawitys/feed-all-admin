<template>
    <div id="Reviews">
        <!--Review TABLE-->
       <v-container fluid>
            <v-layout row justify-center>
            <v-flex md11 xs12>
                <v-card>
                    <v-card-title class="headline pink darken-2 white--text">People (Donees)</v-card-title>
                    <v-data-table v-bind:headers="tableHeaders" :items="reviews" hide-actions class="elevation-1">
                        <template slot="items" slot-scope="props">
                            <td class="text-xs-right"><img :src="props.item.picture" class="profile-preview"></td>
                            <td class="text-xs-right">{{ props.item.level }}</td>
                            <td class="text-xs-right">{{ props.item.name }}</td>
                            <td class="text-xs-right">{{ props.item.cardnumber }}</td>
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
                                    <v-text-field textarea name="cardnumber" label="cardnumber" v-model="newReview.cardnumber" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field type="number" name="level" label="level" v-model="newReview.level" required 
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
  name: 'people',
  data () {
    return {
        newReviewDialog: false,
        valid: true,
        newReview: {
            id:null,
            picture: null,
            name: null,
            cardnumber: null,
            level:null,
            profilePhoto:null
        },
        tableHeaders: [
            {text:"Icon", value: "picture"},
            {text:"level", value: "level"},
            {text:"Name", value: "name"},
            {text:"Card Number", value: "cardnumber"}
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
        db.collection('people').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'picture': doc.data().picture,
                        'profilePath': doc.data().profilePath,
                        'name':doc.data().name,
                        'cardnumber':doc.data().cardnumber,
                        'level':doc.data().level
                    }
                    this.reviews.push(data)
                })
        })
        this.reviewsLoading = false
    },
    resetNewReview(){
        this.newReview= {
            id:null,
            picture: null,
            name: null,
            cardnumber: null,
            level:null,
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
            picture: review.picture,
            profilePath: review.profilePath,
            name: review.name,
            cardnumber: review.cardnumber,
            level: review.level
        }
        this.editOn = true
        this.newReviewDialog =true
    },
    updateReview(){
        if( !this.newReview.name|| !this.newReview.cardnumber || !this.newReview.level)
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
                  const docRef = db.collection('people').doc(this.newReview.id)
                  docRef.set({
                      name: this.newReview.name,
                      cardnumber: this.newReview.cardnumber,
                      level: this.newReview.level,
                      picture: downloadURL,
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
          const docRef = db.collection('people').doc(this.newReview.id)
            docRef.set({
                name: this.newReview.name,
                cardnumber: this.newReview.cardnumber,
                level: this.newReview.level,
                picture: this.newReview.picture,
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
        db.collection('people').doc(review.id).delete()
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
          db.collection('people').add({
                  picture: downloadURL,
                  profilePath: filePath,
                  name: this.newReview.name,
                  cardnumber: this.newReview.cardnumber,
                  level: this.newReview.level
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