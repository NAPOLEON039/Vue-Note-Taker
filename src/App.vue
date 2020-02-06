<template>
  <div id="app">
    <div id="notes-input">
      <h1>Notes</h1>
      <div id="new-notes">
        <input v-model="title" type="text" name="title" id="note-title" placeholder="Enter title here"> <br/>
        <textarea v-model="content" name="content" id="note-content" cols="40" rows="8" placeholder="Type note here"></textarea> 
      </div>
      <button @click="submitNote" id="add-note">Add note</button>
    </div>
    <div id="notes-display">
      <p>Your notes:</p>
      <ul id="notes">
        <li :key="index" v-for="(note, index) in sortedNotes"> 
          <h3><button @click="deleteNote(note.title)">x</button> {{note.title}}</h3>
          <p>{{note.content}}</p>
        </li>
      </ul>
    </div>
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
      // query.docs.map(doc => console.log(doc.data().title))
      query.forEach(doc => {
        // console.log(doc.id)
        const dbNote = {
          title: doc.data().title,
          content: doc.data().content
        }
        this.notes.push(dbNote)
      })
    })
  },
  computed: {
    sortedNotes() {
      return this.notes.sort((a, b) => (a.title > b.title) ? 1 : -1) 
    }
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
      this.notes = this.notes.sort((a, b) => (a.title > b.title) ? 1 : -1)
      console.log('Sorted')
    },
    deleteNote(title) {
      let docToDeleteId = ''
      this.notes = this.notes.filter(note => {
        return note.title !== title
      })
      
      db.collection('notes').where("title", "==", title).get().then(query => {
        query.docs.map(doc => {
          docToDeleteId = doc.id;
        })
        db.collection('notes').doc(docToDeleteId).delete()
      })
    }
   }
}
</script>

<style lang="scss">
$input-height: 50%;
$button-color: rgb(79, 131, 226);
$button-outline: rgb(134, 170, 236);

#notes-input {
  position: absolute;
  left: 0;
  top: 0;
  height: $input-height; 
  width: 100%;
  background-color: rgb(197, 112, 223);
  text-align: center;
}

h1 {
  font-size: 3rem;
}

#note-title {
  margin-bottom: 10px;
}

#add-note {
  background-color: $button-color;
  color: white;
  border: 0.5px solid $button-color;
  border-radius: 5px;
  padding: 5px 15px;
  margin-top: 5px;
  font-size: 1em;
}

#add-note:focus {
  outline-style: none;
  box-shadow: 0 0 1pt 3pt $button-outline;
  // outline-offset: 5px;
}

#notes-display {
  position: absolute;
  top: calc(#{$input-height} + 10px);
}

#notes {
  li {
    white-space: pre-wrap;
    p {
      word-wrap: break-word;
      padding-right: 25px;
    }
  }
}
</style>
