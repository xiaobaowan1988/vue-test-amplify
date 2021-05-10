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
    </div>
  </div>
  <amplify-sign-out></amplify-sign-out>
 </amplify-authenticator>
</template>

<script>
import { API } from 'aws-amplify';
import { createNote } from './graphql/mutations';
import { listNotes } from './graphql/queries';
import { onCreateNote } from './graphql/subscriptions';

export default {
  name: 'App',
  async created() {
    this.getNotes();
  },
  data(){
    return {
      name: '',
      description: '',
      notes: []
    }
  },
  components: {
  },
  methods: {
    async createNote() {
      const { name, description } = this;
      if (!name || !description) return;
      const note = { name, description };
      await API.graphql({
        query: createNote,
        variables: {input: note},
      });
      this.name = '';
      this.description = '';
    },
    async getNotes() {
      const notes = await API.graphql({
        query: listNotes
      });
      this.notes = notes.data.listNotes.items;
    },
    subscribe() {
      API.graphql({ query: onCreateNote })
        .subscribe({
          next: (eventData) => {
            let note = eventData.value.data.onCreateNote;
            if (this.notes.some(item => item.name === note.name)) return; // remove duplications
            this.notes = [...this.notes, note];
          }
        });
    }
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
