<template>
  <div id="app">
    <img src="./assets/logo.png"><br>
    <div>
      user: {{user.phoneNumber}} <br>
      phoneNumber: <input type="text" v-model="phoneNumber">
      <button @click="getCode">Get Code</button>
      <center>
        <div id="recaptcha-container"></div>
      </center>
      code: <input type="text" v-model="code">
      <button @click="submitCode">Submit Code</button>
      <button @click="signOut">Sign Out</button>
    </div>
    <div>
      <div id="firebaseui-auth-container"></div>
    </div>
    <!-- <firebase-ui></firebase-ui> -->
  </div>
</template>

<script>
import firebaseui from 'firebaseui'
import firebase from 'firebase'
var config = {
  apiKey: 'AIzaSyDuUgxopkhZWGwCYAZxY-GHV6HYgIBwwsk',
  authDomain: 'vue-router-exam.firebaseapp.com',
  databaseURL: 'https://vue-router-exam.firebaseio.com',
  projectId: 'vue-router-exam',
  storageBucket: 'vue-router-exam.appspot.com',
  messagingSenderId: '1082986330557'
}
firebase.initializeApp(config)
var uiConfig = {
  signInSuccessUrl: 'http://localhost:8080/',
  signInOptions: [
    {
      provider: firebase.auth.PhoneAuthProvider.PROVIDER_ID,
      recaptchaParameters: {
        size: 'invisible'
      }
    }
  ],
  tosUrl: 'http://localhost:8080/'
}

var ui = new firebaseui.auth.AuthUI(firebase.auth())
ui.start('#firebaseui-auth-container', uiConfig)

export default {
  name: 'app',
  data () {
    return {
      phoneNumber: '',
      code: '',
      user: ''
    }
  },
  mounted () {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.user = user
      }
    })
    window.recaptchaVerifier = new firebase.auth.RecaptchaVerifier('recaptcha-container')
  },
  methods: {
    getCode () {
      var appVerifier = window.recaptchaVerifier
      firebase.auth().signInWithPhoneNumber(this.phoneNumber, appVerifier).then(function (confirmationResult) {
        window.confirmationResult = confirmationResult
      }).catch(function (error) {
        console.log('error', error)
      })
    },
    submitCode () {
      var credential = firebase.auth.PhoneAuthProvider.credential(window.confirmationResult.verificationId, this.code)
      firebase.auth().signInWithCredential(credential).then(function (result) {
        console.log('result', result)
      }).catch(function (error) {
        console.log(error)
      })
    },
    signOut () {
      firebase.auth().signOut().then(() => {
        console.log('sign out')
        this.user = ''
      }).catch(function (error) {
        console.log(error)
      })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
