<template>
  <div id="app">
    <h1>Notes</h1>
    <input v-model="title" type="text" name="title" id="note-title" placeholder="Enter title here"> <br/>
    <textarea v-model="content" name="content" id="note-content" cols="40" rows="8" placeholder="Type note here"></textarea> <button @click="submitNote" id="submit-note">Add note</button>
    <p>Your notes:</p>
    <ul id="notes">
      <li :key="index" v-for="(note, index) in notes"> 
        <h3>{{note.title}}</h3>
        <p>{{note.content}}</p>
      </li>
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
      date: '',
      notes: []
    }
  },
  mounted() {
    db.collection('notes').get().then(query => {
      query.forEach(doc => {
        const dbNote = {
          title: doc.data().title,
          content: doc.data().content
        }
        this.notes.push(dbNote)
      })
    })
    this.sortNotes();
    console.log(this.notes);
    // this.notes.sort((a, b) => a.date.getTime() - b.date.getTime());
  },
  methods: {
    submitNote() {
      if(this.title.length > 0 && this.content.length > 0) {
        db.collection('notes').add({
          title: this.title,
          content: this.content
        })
        .catch(error => console.log(error))
        this.notes.push({
          title: this.title,
          content: this.content
        })
        this.title = ''
        this.content = ''
      }
    },
    sortNotes() {
      this.notes = this.notes.sort((a, b) => (a.title > b.title) ? 1 : -1);
      console.log('Sorted')
    }
   }
}
</script>

<style>
#notes li{
  white-space: pre-wrap;
}
</style>
