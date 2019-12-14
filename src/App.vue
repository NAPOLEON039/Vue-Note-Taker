<template>
  <div id="app">
    <h1>Notes</h1>
    <input v-model="title" type="text" name="title" id="note-title" placeholder="Enter title here"> <br/>
    <textarea v-model="content" name="content" id="note-content" cols="40" rows="8" placeholder="Type note here"></textarea> <button @click="submitNote" id="submit-note">Add note</button>
    <p>Your notes:</p>
    <ul id="notes">
      <li :key="index" v-for="(note, index) in notes"> {{note.title}} - {{note.content}} </li>
    </ul>
  </div>
</template>

<script>
import db from './firebaseInit';

export default {
  name: 'app',
  data() {
    return {
      title: '',
      content: '',
      notes: []
    }
  },
  methods: {
    submitNote() {
      db.collection('notes').add({
        title: this.title,
        content: this.content
      })
      // .then(docRef => {
      //   this.notes.push({
      //     title: this.title,
      //     content: this.content
      //   })
      // })
      .catch(error => console.log(error))
      this.notes.push({
        title: this.title,
        content: this.content
      })
      this.title = ''
      this.content = ''
    }
   }
}
</script>

<style>
#notes li{
  white-space: pre;
}
</style>
