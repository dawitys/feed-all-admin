<template>
    <div>
    <div id="left" class="box">
    <div class="box__header">
      <h2 class="box__header-title">Total Balance</h2>
    </div>
    <div class="box__body">
      <div class="stats stats--main">
        <div class="stats__amount">{{balance}}</div>
        <div class="stats__caption">Ethiopian Birr</div>
        <div class="stats__change">
          <div class="stats__value stats__value--positive">+6%</div>
          <div class="stats__period">this week</div>
        </div>
      </div>
    </div>
  </div>
    <div id="ServiceProviders">
        <!--Service TABLE-->
        <v-container fluid>
            <v-layout row justify-center>
            <v-flex md11 xs12>
                <v-card>
                    <v-card-title class="headline brown darken-2 white--text">ServiceProviders</v-card-title>
                    <v-data-table v-bind:headers="tableHeaders" :items="serviceProviders" hide-actions class="elevation-1">
                        <template slot="items" slot-scope="props">
                            <td class="text-xs-right">{{ props.item.name }}</td>
                            <td class="text-xs-right">{{ props.item.location }}</td>
                            <td class="text-xs-right">{{ props.item.paid }}</td>
                            <td class="text-xs-right">{{ props.item.unpaid }}</td>
                            <td class="text-xs-right">
                                <!-- <v-btn icon  @click="startEdit(props.item)"><v-icon>fa-pencil</v-icon></v-btn> -->
                                <v-btn icon  @click="deleteService(props.item)"><v-icon>fa-money</v-icon><i> Pay</i></v-btn>
                            </td>
                        </template>
                    </v-data-table>
                </v-card>
            </v-flex>
        </v-layout>
        </v-container>
        <v-btn color="brown darken-2" dark fixed bottom right fab @click.native.stop="newServiceDialog = !newServiceDialog">
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
                                    <v-text-field name="name" label="name" v-model="newService.name" required 
                                    :rules="[v => !!v || 'Item is required']"></v-text-field>
                                    <v-text-field textarea name="location" label="location" v-model="newService.location" required 
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
    </div>
</template>

<script>
import firebase from './firebaseInit'
import 'firebase/firestore'
const db = firebase.firestore()
export default {
  name: 'ServiceProviders',
  data () {
    return {
        newServiceDialog: false,
        serviceFormValid: true,
        newService: {
            id:null,
            icon: null,
            name: null,
            location: null,
            position:null
        },
        tableHeaders: [
            {text:"Name", value: "name"},
            {text:"Location", value: "location"},
            {text:"Paid", value: "paid"},
            {text:"Unpaid", value: "Unpaid"}
        ],
        serviceProviders: [],
        balance : 10000,
        serviceProvidersLoading:false,
        editOn:false
    }
  },
  methods:{
    listServiceProviders(){
        this.serviceProvidersLoading = true
        this.serviceProviders=[]
        db.collection('clients').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'id': doc.id,
                        'name':doc.data().name,
                        'location':doc.data().location,
                        'paid': doc.data().paid,
                        'unpaid':doc.data().unpaid
                    }
                    this.serviceProviders.push(data)
                })
        })
        db.collection('balance').get().then(querySnapshot => {
            querySnapshot.forEach(doc => {
                const data = {
                    'amount':doc.data().amount,
                }
                this.balance = data.amount
            })
        })
        this.serviceProvidersLoading = false
    },
    createNewService(){
        if( !this.newService.icon || !this.newService.name|| !this.newService.location || !this.newService.position)
            return
        db.collection('clients').add({
            paid: this.newService.paid,
            title: this.newService.name,
            location: this.newService.location,
            position: this.newService.position
        })
        .then(docRef => {
            this.listServiceProviders()
            this.resetNewService()
            this.newServiceDialog = false
        })
        .catch(error => console.log(err))
    },
    resetNewService(){
        this.newService= {
            id:null,
            icon: null,
            name: null,
            location: null,
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
            name: service.name,
            location: service.location,
            position: service.position
        }
        this.editOn = true
        this.newServiceDialog =true
    },
    updateService(){
        if( !this.newService.icon || !this.newService.name|| !this.newService.location || !this.newService.position)
            return
        const docRef = db.collection('clients').doc(this.newService.id)
        docRef.set({
            name: this.newService.name,
            location: this.newService.location,
            paid: this.newService.paid,
            unpaid: this.newService.unpaid
            })
            .then( (doc)=> {
                this.listServiceProviders()
                console.log("Document successfully written!", doc);
                this.resetNewService()
            })
            .catch(()=> {
                console.error("Error writing document: ", error);
        })
    },
    deleteService(service){
        if(confirm("You are about to make payment. Are you sure?")){
            const dRef = db.collection('balance').doc("amount")

            db.collection('balance').get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    const data = {
                        'amount':doc.data().amount,
                    }
                    this.balance = data.amount
                })
            })
            dRef.update({amount : this.balance - service.unpaid})

            const docRef = db.collection('clients').doc(service.id)
            docRef.update({paid : service.paid+service.unpaid})
            docRef.update({unpaid : 0})
        .then(data => {
            this.listServiceProviders()
            console.log("Payment made!");
        }).catch(error => {
            console.error("Error making payment : ", error);
        })
      }
    }
  },
  created(){
      this.listServiceProviders()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.box {
	background: linear-gradient(#2b9966, #083110);
	margin: 15px auto 0;
	border-radius: 5px;
    width: 500px;
    position: relative;
}

.box__header {
	padding: 15px 25px;
	position: relative;
}

.box__header-title {
	color: #fff;
	font-size: 18px;
}

.box__body {
	padding: 0 25px;
}

/* STATS */

.stats {
	color: #fff;
	position: relative;
	padding-bottom: 25px;
}

.stats__amount {
	font-size: 42px;
	font-weight: bold;
	line-height: 1.2;
}

.stats__caption {
	font-size: 18px;

}

.stats__change {
	position: absolute;
	top: 10px;
	right: 0;
	text-align: right;
	color: #B1B7C8;
}

.stats__value {
	font-size: 18px;
}

.stats__period {
	font-size: 14px;
}

.stats__value--positive {
	color: #AEDC6F;
}

.stats__value--negative {
	color: #FB5055;
}

.stats--main .stats__amount {
	font-size: 54px;
}

</style>
