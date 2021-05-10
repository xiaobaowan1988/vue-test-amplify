<template>
 <amplify-authenticator>
  <div id="app">
    <h1>Note App</h1>
    <input type="text" v-model="name" placeholder="Note name">
    <input type="text" v-model="description" placeholder="Note description">
    <button v-on:click="createNote">Create Note</button>
    <div v-for="item in notes" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
      <p>{{ item.done }}</p>
      <br/>
    </div>
  </div>
  <amplify-sign-out></amplify-sign-out>
 </amplify-authenticator>
</template>

<script>
//import { API } from 'aws-amplify';
//import { createNote } from './graphql/mutations';
//import { listNotes } from './graphql/queries';
//import { onCreateNote } from './graphql/subscriptions';
import { DataStore } from  '@aws-amplify/datastore';
import { Note } from '@/models';

export default {
  name: 'App',
  data(){
    return {
      name: '',
      description: '',
      notes: [],
      done: '',
      subscription: undefined
    }
  },
  components: {
  },
  created: function(){
    //Subscribe to changes
    this.subscription = DataStore.observe(Note).subscribe(msg => {
      console.log(msg.model, msg.opType, msg.element);
      this.listNotes();
    });
  }, 
  destroyed() {
    if (!this.subscription) return;
    this.subscription.unsubscribe();
  },
  methods: {
   async  createNote() {
     const note = await DataStore.save(
         new Note({
           name: this.name,
           description: this.description,
           done: false
         })
      );
      console.log(note)
      this.name = '';
      this.description = '';
    },
   async  listNotes() {
      this.notes = await DataStore.query(Note);
      console.log(JSON.stringify(this.notes,null,2))
    },
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
