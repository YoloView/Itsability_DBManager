<template>
    <div v-if="anone === true">
        <h1>Login</h1>
        <div id="firebaseui-auth-container"></div>
    </div>
    <div v-else>
        <h1>{{ this.displayName }}님 반갑습니다!</h1>
        <button v-on:click="Logout">Logout</button>
    </div>
</template>

<script>
import * as firebaseui from 'firebaseui'
import * as firebase from 'firebase/app'
import 'firebase/firestore'
import {config} from '../../firebase_api'

firebase.initializeApp(config)
const auth = firebase.auth()
const db = firebase.firestore()
// const storage = firebase.storage()
const ui = new firebaseui.auth.AuthUI(auth)

export default {
  data: function () {
    return {
      anone: true,
      uid: '',
      displayName: '',
      email: ''
    }
  },
  methods: {
    initUI: function () {
      ui.start('#firebaseui-auth-container', {
        signInOptions: [{
          provider: firebase.auth.GoogleAuthProvider.PROVIDER_ID,
          authMethod: 'https://accounts.google.com',
          clientId: '659989827911-u4e6bnt0n77360n4nabugbf3il150trj.apps.googleusercontent.com'
        }],
        callbacks: {
          signInSuccessWithAuthResult: (authResult, redirectUrl) => {
            alert(`${authResult.user.displayName} login 성공!`)
            return false
          }
        }
      })
    },
    Loging: function () {
      let userRef = db.collection('user').doc('ManagePermission') // .where(this.anone, '==', true)
      let vaildUser = userRef.get()
        .then((snapshot) => {
          return (snapshot.get(this.uid))
        })
        .catch((err) => {
          console.log('Error getting documents', err)
        })
      if (vaildUser) {
        this.anone = false
      } else {
        alert('미인증된 유저입니다. 자세한 내용은 정훈이에게 물어보세요.')
      }
    },
    Logout: function () {
      auth.signOut().then(() => {
        this.uid = ''
        this.displayName = ''
        this.email = ''
        this.anone = true
      }).catch(err => {
        alert(err)
        console.log(err)
      }).then(() => {
        window.location.reload()
      })
    }
  },

  mounted: function () {
    auth.onAuthStateChanged(user => {
      if (user) {
        this.uid = user.uid
        this.displayName = user.displayName
        this.email = user.email
        this.Loging()
      } else {
        this.initUI()
      }
    })
  }
}
</script>
