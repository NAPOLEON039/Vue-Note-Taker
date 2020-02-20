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
          <button @click="deleteNote(note.title)">x</button> <h3 class="title-display">{{note.title}}</h3> <!-- Use the update method to edit notes. It's under Add data in the docs -->
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
$input-height: 70vh;
$add-note-button-color: rgb(79, 131, 226);
$add-button-outline: rgb(134, 170, 236);
$delete-note-button-color: rgb(211, 40, 40);
$delete-note-button-outline: rgb(245, 138, 138);

#notes-input {
  position: absolute;
  left: 0;
  top: 0;
  height: $input-height; 
  width: 100%;
  background-color: rgb(197, 112, 223);
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

h1 {
  font-size: 3rem;
}

#note-title {
  margin-bottom: 10px;
}

#add-note {
  align-self: center;
  background-color: $add-note-button-color;
  color: white;
  width: 100px;
  border: 0.5px solid $add-note-button-color;
  border-radius: 5px;
  padding: 5px 15px;
  margin-top: 5px;
  font-size: 1em;
}

#add-note:focus {
  outline-style: none;
  box-shadow: 0 0 1pt 3pt $add-button-outline;
  // outline-offset: 5px;
}

#notes-display {
  position: absolute;
  top: calc(#{$input-height} + 10px);
}

#notes-display > p {
  margin-left: 20px;
  font-size: 1.2em;
}

#notes {
  list-style: none;
  li {
    white-space: pre-wrap;
    .title-display {
      display: inline-block;
    }
    button {
      background-color: $delete-note-button-color;
      color: white;
      width: 25px;
      height: 30px;
      border: 0.5px solid $delete-note-button-color;
      border-radius: 5px;
      margin-right: 15px;
    }
    button:focus {
      outline-style: none;
      box-shadow: 0 0 1pt 1.5pt $delete-note-button-outline;
    }
    p {
      word-wrap: break-word;
      padding-right: 25px;
    }
  }
}
</style>
