<template>
  <div id="Posts" class="container">
    <form>
      <div class="row">
        <div class="col s12 m12">
          <div class="card">
            <div class="card-content">
              <div class="row">
                <div class="input-field col s12">
                  <input v-model="title" id="title" type="text" placeholder="Post Title" class="validate" required>
                </div>
              </div>
              <div class="row">
                <div class="input-field col s12 m12">
                  <quill-editor ref="myTextEditor"
                          v-model="body"
                          :options="editorOptions" required>
                  </quill-editor>
                </div>
              </div>
            </div>
            <div class="card-action right-align">
              <button class="pull-left waves-effect waves-light btn pointer indigo darken-1" type="button" @click="listPhotos"><span v-if="!photosOn">Photos</span><span v-if="photosOn">Hide</span></button>
              <button class="waves-effect waves-teal btn-flat pointer" type="button" @click="cancelPost">Cancel</button>
              <button v-if="!editOn" class="waves-effect waves-light btn pointer" type="submit" @click="createPost">Post</button>
              <button v-if="editOn" class="waves-light btn pointer" type="submit" @click="updatePost">Update</button>
            </div>
          </div>
        </div>
      </div>
    </form>
     <div class="row">
        <div class="col s12">
           <div class="card">
        <div class="card-content align-center" v-if="photosOn">
          <div class="row">
            <div class="col s12 m3" v-for="photo in postPhotos" v-bind:key="photo.id">
                <div class="limited-height">
                  <img :src="photo.url" class="fit pointer" title="Click to add to form" @click="appendPhoto(photo.url)">
                </div>
            </div>
          </div>
        </div>
    </div>
        </div>
      </div>
   
    <div class="row">
      <div class="col s12">
        <div class="card extra-padding">
          <table class="responsive-table highlight">
            <thead><tr><th>CreatedAt</th><th>Title</th><th>Body</th><th>Options</th></tr></thead>
            <tbody>
                <tr v-for="post in posts" v-bind:key="post.id">
                      <td>{{post.createdAt.toString()}}</td>
                      <td>{{post.title}}</td>
                      <td>{{post.body}}</td>
                      <td><i class="fa fa-pencil pointer" @click="startEdit(post)"></i> <i class="fa fa-times pointer" @click="deletePost(post.id)"></i></td>
                </tr>
              </tbody>
        </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from './firebaseInit'
import 'firebase/firestore'
import { quillEditor } from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
const db = firebase.firestore()

export default {
  name: 'Posts',
  components: {
    quillEditor
   },
  data () {
    return {
      editorOptions:{
      },
      posts:[],
      title: null,
      body:null,
      createdAt: null,
      docId:null,
      editOn:false,
      postPhotos:[],
      photosOn:false,
      postsLoading:false,
      listingPhotos:false
    }
  },
  methods:{
    listPosts(){
      this.postsLoading = true
      this.posts=[]
      db.collection('posts').orderBy('createdAt').get().then(querySnapshot => {
              querySnapshot.forEach(doc => {
                  const data = {
                      'id': doc.id,
                      'title': doc.data().title,
                      'body':doc.data().body,
                      'createdAt':doc.data().createdAt
                  }
                  this.posts.push(data)
              })
      })
      this.postsLoading = false
    },
    createPost(){
      if(!this.body || !this.title) return
      db.collection('posts').add({
        title: this.title,
        body: this.body,
        createdAt: new Date()
      })
      .then(docRef => {
        this.listPosts()
        this.cancelPost()
      })
      .catch(error => console.log(err))
    },
    cancelPost(){
      this.title=null,
      this.body=null
      this.editOn = false,
      this.createdAt= null,
      this.docId=null
    },
    startEdit(post){
      this.editOn=true
      this.title=post.title
      this.body=post.body
      this.createdAt=post.createdAt
      this.docId=post.id
    },
    updatePost(){
      const docRef = db.collection('posts').doc(this.docId)
      console.log(docRef)
      docRef.set({
          title: this.title,
          body: this.body,
          createdAt: this.createdAt,
          editedAt: new Date()
        })
        .then( (doc)=> {
            this.listPosts()
            console.log("Document successfully written!", doc);
            this.cancelPost()
            
        })
        .catch(()=> {
            console.error("Error writing document: ", error);
      })
    },
    deletePost(postId){
      if(confirm("Are you sure?")){
        const docRef = db.collection('posts').doc(postId).delete()
        .then(data => {
            this.listPosts()
            console.log("Document successfully deleted!");
        }).catch(error => {
            console.error("Error removing document: ", error);
        })
      }
    },
    listPhotos(){
      
      this.photosOn=!this.photosOn
      if(this.listingPhotos) return
      this.listingPhotos=true
      if(this.postPhotos.length>0) return
      db.collection("postPhotos").get().then(querySnapshot => {
                  querySnapshot.forEach(doc => {
                      const postPhoto = {
                          'id': doc.id,
                          'name': doc.data().name,
                          'path':doc.data().path,
                          'url':doc.data().url
                      }
                      this.postPhotos.push(postPhoto)
                     // console.log("photos", postPhoto)
                  });
      });  
    },
    hidePhotos(){
      this.photosOn=false
    },
    appendPhoto(url){
      if(!this.body)
      this.body = '<img src="'+url+'"/>'
      else
        this.body = this.body+ '<img src="'+url+'"/>'
    },
    appendTemplate(tmpl){
      if(!this.body)
      this.body = tmpl
      else
        this.body = this.body+ tmpl
    }
  },
  created(){
    this.listPosts()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.pointer{
  cursor:pointer;
}
.pull-left{
  float:left;
}
.limited-height{
    height:200px;
    overflow:hidden;
    margin: 15px;
}
.fit{
  max-height:200px;
  max-width:100%;
}
textarea.materialize-textarea{height: 200px;}
</style>
