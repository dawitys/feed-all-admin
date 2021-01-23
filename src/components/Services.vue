<template>
    <div id="Services">
        <!--Service TABLE-->
        <v-container fluid>
            <v-layout row justify-center>
            <v-flex md11 xs12>
                <v-card>
                    <v-card-title class="headline pink darken-2 white--text">Services</v-card-title>
                    <v-data-table v-bind:headers="tableHeaders" :items="services" hide-actions class="elevation-1">
                        <template slot="items" slot-scope="props">
                            <td class="text-xs-right">{{ props.item.position }}</td>
                            <td class="text-xs-right"><v-icon>{{ props.item.icon }}</v-icon></td>
                            <td class="text-xs-right">{{ props.item.title }}</td>
                            <td class="text-xs-right">{{ props.item.description }}</td>
                            <td class="text-xs-right">
                                <v-btn icon  @click="startEdit(props.item)"><v-icon>fa-pencil</v-icon></v-btn>
                                <v-btn icon  @click="deleteService(props.item)"><v-icon>fa-times</v-icon></v-btn>
                            </td>
                        </template>
                    </v-data-table>
                </v-card>
            </v-flex>
        </v-layout>
        </v-container>
        <v-btn color="pink darken-2" dark fixed bottom right fab @click.native.stop="newServiceDialog = !newServiceDialog">
            <v-icon>fa-plus</v-icon>
        </v-btn>
        <!--NEW SERVICE MODAL-->
            <v-layout row justify-center>
            <v-dialog  v-model="newServiceDialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                <span v-if="!editOn" class="headline">Create New Service</span>
                <span v-else class="headline">Edit Service</span>
                </v-card-title>
                <v-card-text>
                    <v-container grid-list-md>
                        <v-layout wrap>
                            <v-flex xs12>
                                <v-form v-model="serviceFormValid" ref="serviceForm" lazy-validation>
                                    <v-text-field name="icon" label="icon" v-model="newService.icon" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field name="title" label="title" v-model="newService.title" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field textarea name="description" label="description" v-model="newService.description" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field type="number" name="position" label="position" v-model="newService.position" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                </v-form>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" flat @click.native="resetNewService()">Cancel</v-btn>
                <v-btn v-if="!editOn" color="teal darken-1" flat @click.native="createNewService()" :disabled="!serviceFormValid">Submit</v-btn>
                <v-btn v-else color="teal darken-1" flat @click.native="updateService()" :disabled="!serviceFormValid">Submit</v-btn>
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
  name: 'Services',
  data () {
    return {
        newServiceDialog: false,
        serviceFormValid: true,
        newService: {
            id:null,
            icon: null,
            title: null,
            description: null,
            position:null
        },
        tableHeaders: [
            {text:"Position", value: "position"},
            {text:"Icon", value: "icon"},
            {text:"Title", value: "title"},
            {text:"Description", value: "description"}
        ],
        services: [],
        servicesLoading:false,
        editOn:false
    }
  },
  methods:{
    listServices(){
        this.servicesLoading = true
        this.services=[]
        db.collection('services').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'icon': doc.data().icon,
                        'title':doc.data().title,
                        'description':doc.data().description,
                        'position':doc.data().position
                    }
                    this.services.push(data)
                })
        })
        this.servicesLoading = false
    },
    createNewService(){
        if( !this.newService.icon || !this.newService.title|| !this.newService.description || !this.newService.position)
            return
        db.collection('services').add({
            icon: this.newService.icon,
            title: this.newService.title,
            description: this.newService.description,
            position: this.newService.position
        })
        .then(docRef => {
            this.listServices()
            this.resetNewService()
            this.newServiceDialog = false
        })
        .catch(error => console.log(err))
    },
    resetNewService(){
        this.newService= {
            id:null,
            icon: null,
            title: null,
            description: null,
            position:null
        }
        this.editOn=false
        this.newServiceDialog=false
        this.$refs.serviceForm.reset()
    },
    startEdit(service){
        this.newService= {
            id:service.id,
            icon: service.icon,
            title: service.title,
            description: service.description,
            position: service.position
        }
        this.editOn = true
        this.newServiceDialog =true
    },
    updateService(){
        if( !this.newService.icon || !this.newService.title|| !this.newService.description || !this.newService.position)
            return
        const docRef = db.collection('services').doc(this.newService.id)
        docRef.set({
            title: this.newService.title,
            description: this.newService.description,
            position: this.newService.position,
            icon: this.newService.icon
            })
            .then( (doc)=> {
                this.listServices()
                console.log("Document successfully written!", doc);
                this.resetNewService()
            })
            .catch(()=> {
                console.error("Error writing document: ", error);
        })
    },
    deleteService(service){
        if(confirm("Are you sure?")){
        const docRef = db.collection('services').doc(service.id).delete()
        .then(data => {
            this.listServices()
            console.log("Document successfully deleted!");
        }).catch(error => {
            console.error("Error removing document: ", error);
        })
      }
    }
  },
  created(){
      this.listServices()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
