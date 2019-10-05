<template>
    <div v-if="anone === true">
        <h1>Login</h1>
        <div id="firebaseui-auth-container"></div>
    </div>
    <div v-else>
        <h1>{{ this.displayName }}님 반갑습니다!</h1>
        <button v-on:click="Logout">Logout</button>
        <template id="grid-template">
        </template>
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

<style>
body {
  font-family: Helvetica Neue, Arial, sans-serif;
  font-size: 14px;
  color: #444;
}

table {
  border: 2px solid #42b983;
  border-radius: 3px;
  background-color: #fff;
}

th {
  background-color: #42b983;
  color: rgba(255,255,255,0.66);
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

td {
  background-color: #f9f9f9;
}

th, td {
  min-width: 120px;
  padding: 10px 20px;
}

th.active {
  color: #fff;
}

th.active .arrow {
  opacity: 1;
}

.arrow {
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  margin-left: 5px;
  opacity: 0.66;
}

.arrow.asc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-bottom: 4px solid #fff;
}

.arrow.dsc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid #fff;
}
</style>
