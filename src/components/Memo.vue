<template>
  <div class="container mt-3 mb-3">
    <div id="app">
      <div class="menu">
        <a href="#" @click="create">new</a> |
        <a href="#" @click="destroy(docId)">delete</a>
      </div>
      <hr>
      <p v-if="!docId">↑ please click above "new" link to start your memo.</p>
      <div v-if="docId">
        <div class="title mb-3">
          <h3 id="title">title</h3>
          <ul class="list-unstyled mb-0" v-for="(task, k) in tasks" :key="task.id">
            <li class="border-bottom" v-for="(content, id) in task" :key="content.id" @click="showContent(content, id, k)" v-bind:class="{activeList : k === nthMemo && id === docId}">
              {{content | firstLine}}
            </li>
          </ul>
        </div>

        <div class="detail">
          <h3 id="detail">detail</h3>
          <textarea class="form-control mb-3" v-model="detail" rows="5"></textarea>
          <button @click="update(docId)" class="btn btn-primary">update detail</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase/app'
import 'firebase/app'
import 'firebase/firestore'
import 'firebase/analytics'

export default {
  name: 'Memo',
  data: function() {
    return {
      detail: '',
      tasks: [],
      nthMemo: null,
      docId: null,
      db: firebase.firestore()
    }
  },
  beforeCreate () {
    const firebaseConfig = {
      apiKey: "process.env.VUE_APP_API_KEY",
      authDomain: process.env.VUE_APP_PROJECT_ID + ".firebaseapp.com",
      databaseURL: "https://" + process.env.VUE_APP_PROJECT_ID + ".firebaseio.com",
      projectId: process.env.VUE_APP_PROJECT_ID,
      storageBucket: process.env.VUE_APP_PROJECT_ID + ".appspot.com",
      messagingSenderId: process.env.VUE_APP_SENDER_ID,
      appId: process.env.VUE_APP_APP_ID,
      measurementId: process.env.VUE_APP_MEASUREMENT
    }
    firebase.initializeApp(firebaseConfig)
    firebase.analytics()
  },
  created () {
    this.read()
  },
  methods: {
    async read () {
      const promise = new Promise((resolve) => {
        this.db.collection("tasks").orderBy("timestamp","desc").onSnapshot((querySnapshot) => {
          this.tasks = []
          querySnapshot.forEach((doc) => {
            const task = {}
            for (const field in doc.data()) {
              if (field === "content") {
                task[doc.id] = doc.data()[field]
              }
            }
            this.tasks.push(task)
          })
          resolve()
        })
      })
      await promise
      this.selectFirstMemo()
    },
    create () {
      this.db.collection("tasks").add({
        content: "新規メモ",
        timestamp: firebase.firestore.Timestamp.now()
      })
      .then((docRef) => {
          console.log("Document written with ID: ", docRef.id)
          this.selectFirstMemo()
      })
      .catch(function(error) {
          console.error("Error adding document: ", error)
      })
    },
    update (docId) {
      window.location.hash = ""
      if (!this.docId) {
        console.log("Memo has not selected.")
        return
      }
      return this.db.collection("tasks").doc(docId).update({
        content: this.detail,
        timestamp: firebase.firestore.Timestamp.now()
      })
      .then(() => {
          console.log("Document successfully updated!")
          this.selectFirstMemo()
          window.location.hash = "title"
      })
      .catch(function(error) {
          console.error("Error updating document: ", error)
      })
    },
    destroy (docId) {
      if (!this.docId) {
        console.log("Memo has not selected.")
        return
      }
      this.db.collection("tasks").doc(docId).delete()
      .then(() => {
        console.log("Document successfully deleted!")
        this.docId = null
        this.detail = ''
        this.selectFirstMemo()
      })
      .catch(function(error) {
          console.error("Error removing document: ", error)
      })
    },
    selectFirstMemo () {
      try {
        const firstMemo = this.tasks[0]
        const firstDocId = Object.keys(firstMemo)[0]
        const firstDocIdContent = firstMemo[firstDocId]

        this.nthMemo = 0
        this.docId = firstDocId
        this.detail = firstDocIdContent
      } catch(e) {
        console.log("There are no memos yet. : " + e)
      }
    },
    showContent (content, id, k) {
      this.detail = content
      this.nthMemo = k
      this.docId = id
    }
  },
  filters: {
    firstLine (value) {
      const firstLineMaxLength = 40
      return value.split("\n")[0].substr(0, firstLineMaxLength)
    }
  }
}
</script>

<style>
.title {
  max-height: 450px;
  overflow: scroll;
}
body {
   background: linear-gradient(90deg, #FFFFFD, #F2F2F0);
}
.activeList {
  background-color: #E3E3E3;
  padding: 0 0.3rem 0
}
</style>
