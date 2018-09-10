<template>
  <div id="app">
  <Notebook @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index" />
  <Page @save-page="savePage" @delete-page="deletePage" :page="pages[index]" />
  </div>
</template>

<script>
import Notebook from "./components/Notebook"
import Page from "./components/Page"
import Firebase from "firebase"



  // Initialize Firebase
  const config = {
    apiKey: "AIzaSyAs_Z19X56Y4RLipX-ECRFVu3MZ6eAbaQc",
    authDomain: "notebook-d932b.firebaseapp.com",
    databaseURL: "https://notebook-d932b.firebaseio.com",
    projectId: "notebook-d932b",
    storageBucket: "",
    messagingSenderId: "720350948673"
  };
  
  let database = Firebase.initializeApp(config).database().ref('notebook');


export default {
  name: 'App',
  components: {
    Notebook,
    Page
  },
  data: () => ({
    pages: [],
    index: 0
  }),
      mounted() {
      database.once('value', (pages) => {
        pages.forEach( (page) => {
          this.pages.push({
            ref: page.ref,
            title: page.child('title').val(),
            content: page.child('content').val()
          })
        })
      })
    },
  methods: {
    newPage () {
      this.pages.push({
        title: '',
        content: ''
      })
      this.index = this.pages.length - 1

    },
    changePage (index) {
      this.index = index
    },
    savePage () {
      let page = this.pages[this.index]

      if (page.ref) {
        this.updateExistingPage(page)
      } else {
        this.insertNewPage(page)
      }
    },
    updateExistingPage (page) {
      page.ref.set({
        title:  page.title,
        content: page.content
      })
    },
    insertNewPage (page) {
      page.ref = database.push(page)
    },
    deletePage () {
      let ref = this.pages[this.index].ref
      ref && ref.remove()
      this.pages.splice(this.index, 1)
      this.index = Math.max(this.index - 1, 0)
    }
  }

}
</script>

<style>
html, body, #app {
  height: 100%;
}

body{
  margin: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  display: flex;
  flex-direction: row;
}
</style>
