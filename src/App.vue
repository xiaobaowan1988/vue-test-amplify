<template>
 <amplify-authenticator>
  <div id="app">
  <v-app id="inspire">
    <v-data-table
      :headers="headers"
      :items="notes"
      sort-by="name"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar
          flat
        >
          <v-toolbar-title>Note CRUD</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="primary"
                dark
                class="mb-2"
                v-bind="attrs"
                v-on="on"
              >
                New Item
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>
  
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="editedItem.name"
                        label="Note name"
                      ></v-text-field>
                      <p>{{editedItem.name}}</p>
                    </v-col>
                 </v-row>
                  <v-row>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="editedItem.description"
                        label="Description"
                      ></v-text-field>
                      <p>{{editedItem.description}}</p>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
  
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="close"
                >
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="save"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          small
          @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
      </template>
    </v-data-table>
  </v-app>
</div>
 </amplify-authenticator>
</template>

<script>
//import { API } from 'aws-amplify';
//import { createNote } from './graphql/mutations';
//import { listNotes } from './graphql/queries';
//import { onCreateNote } from './graphql/subscriptions';
import { DataStore } from  '@aws-amplify/datastore';
import { Note } from '@/models';

export default{
  name: 'App',
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: 'Name',
        align: 'start',
        sortable: false,
        value: 'name',
      },
      { text: 'Description', value: 'description' },
      { text: 'Done', value: 'done' },
      { text: 'Actions', value: 'actions', sortable: false },
    ],
    notes: [],
    editedIndex: -1,
    editedItem: {
      id: '',
      name: '',
      description: '',
      done: false,
    },
    defaultItem: {
      id: '',
      name: '',
      description: '',
      done: false,
    },
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    },
  },

  created () {
    //this.initialize(),
    this.subscription = DataStore.observe(Note).subscribe(msg => {
      console.log(msg.model, msg.opType, msg.element);
      this.listNotes();
    });
  },

  methods: {

    // editItem (item) {
    //   this.editedIndex = this.notes.indexOf(item)
    //   this.editedItem = Object.assign({}, item)
    //   this.dialog = true
    // },


    async editItem(item){
     //const original = await DataStore.query(Note, this.id);
     console.log("before update",item)

     const original = item

     this.editedIndex = this.notes.indexOf(item)
     this.editedItem = Object.assign({}, item)
     await DataStore.save(
       Note.copyOf(original, updated => {
         updated.name = `name ${Date.now()}`;
    })  
  );
    this.dialog = true
  },
  //console.log(this.editedIndex,this.editedItem),
  
  

     async deleteItem (item) {
       console.log(item)
       this.editedIndex = this.notes.indexOf(item)
       this.editedItem = Object.assign({}, item)
       //this.editedItem = item
       this.dialogDelete = true
       console.log(this.editedIndex)
       
     },

    



    async deleteItemConfirm () {
      //this.notes.splice(this.editedIndex, 1)
     
     //console.log("delte item",item)
     //this.editedIndex = this.notes.indexOf(item)
     //this.editedItem = Object.assign({}, item)
     
     //this.dialogDelete = true
     //await DataStore.delete(item);
     //this.closeDelete()
     console.log("notes all",this.notes)
     const deleteItem = this.notes[this.editedIndex]
     await DataStore.delete(deleteItem)
     this.closeDelete()
    },

    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    async save () {
      if (this.editedIndex > -1) {
        Object.assign(this.notes[this.editedIndex], this.editedItem)
      } else {
        //this.notes.push(this.editedItem)
        const note = await DataStore.save(
        new Note({
           name: this.editedItem.name,
           description: this.editedItem.description,
           done: this.editedItem.done
         })
      );
      console.log(note)
      this.name = '';
      this.description = '';
      this.close()
      }
      this.close()
    },

    // async  save() {
    //  const note = await DataStore.save(
    //      new Note({
    //        name: this.name,
    //        description: this.description,
    //        done: this.done
    //      })
    //   );
    //   console.log(note)
    //   this.name = '';
    //   this.description = '';
    //   this.close()
    // },
    async  listNotes() {
      //this.notes = await DataStore.query(Note, Predicates.ALL, {
      //page: 0,
      //limit: 2
      this.notes = await DataStore.query(Note)
      console.log(JSON.stringify(this.notes,null,2))
    }



  },
}
</script>
