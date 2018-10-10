<template>
  <div id="app">
    <header class="header">
      <div class="header__inner">
        <template v-if="isSignedIn && userName && userPic">
          <div class="header__user-image"><img :src='userPic'></div>
          <p class="header__user-name">{{ userName }}</p>
        </template>
        <button class="header__auth-button" @click="authFunction">
          {{ authButtonText }}
        </button>
      </div>
    </header>

    <template v-if="isSignedIn">
      <main>
        <div class="chat-area">
          <div class="conversation">
            <ul class="conversation__inner">
              <li v-for="list in messageList">
                <div class="conversation__user-image"><img :src="list.profilePicUrl" alt=""></div>
                <div class="conversation__col">
                  <p class="conversation__user-name">{{list.name}}</p>
                  <p class="conversation__user-text">{{list.text}}</p>
                </div>
              </li>
            </ul>
          </div>
          <div class="post">
            <input type="text" class="post__text" v-model="message">
            <button type="button" class="post__button" @click="postMessage">SEND</button>
          </div>
        </div>
      </main>
    </template>

  </div>
</template>

<script>

export default {
  name: 'App',
  data () {
    return {
      userName: null,
      userPic: null,
      isSignedIn: null,
      authButtonText: null,
      authFunction: function () {},
      message: null,
      messageList: []
    }
  },
  created () {
    this.onAuthStateChanged()
    this.isUserSignedIn()
    this.loadMessages()
  },
  methods: {
    onAuthStateChanged () {
      let that = this
      firebase.auth().onAuthStateChanged( user => {
        if (user) {
          that.userName = that.getUserName()
          that.userPic = that.getProfilePicUrl()
          that.authButtonText = 'Sign-out'
          that.authFunction = that.signOut
          that.isSignedIn = true
        } else {
          that.userName = null
          that.userPic = null
          that.authButtonText = 'Sign-in with Google'
          that.authFunction = that.signIn
          that.isSignedIn = false
        }
      })
    },
    signIn () {
      let provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithPopup(provider)
    },
    signOut () {
      firebase.auth().signOut()
    },
    isUserSignedIn() {
      this.isSignedIn = !!firebase.auth().currentUser || false
    },
    getProfilePicUrl() {
      return firebase.auth().currentUser.photoURL || '/static/images/profile_placeholder.png';
    },
    getUserName() {
      return firebase.auth().currentUser.displayName;
    },
    postMessage(){
      let that = this
      if (!that.isSignedIn || !that.message) return
      firebase.database().ref('/messages/').push({
        name: that.getUserName(),
        text: that.message,
        profilePicUrl: that.getProfilePicUrl()
      }).catch( error => {
        console.error('Error writing new message to Firebase Database', error);
      });

      that.message = null;
    },
    loadMessages(){
      let that = this
      firebase.database().ref('/messages/').on('value', (snapshot) => {
        if (snapshot) {
          let rootList = snapshot.val();
          let messageList = [];
          Object.keys(rootList).forEach((val, key) => {
            rootList[val].id = val;
            messageList.push(rootList[val]);
          })
          that.messageList = messageList;
        }
      })
    }
  }
}
</script>

<style>
  body{
    background-color: #ECEFF1;
  }
  .header{
    background-color: #009CE5;
  }
  .header__inner{
    width: 90%;
    height: 70px;
    margin: 0 auto;
    display: flex;
    align-items: center;
  }
  .header__user-image{
    position: relative;
    width: 50px;
    height: 50px;
    overflow: hidden;
    border-radius: 50%;
  }
  .header__user-image img{
    width: 100%;
    height: auto;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
  }
  .header__user-name{
    margin: 0 15px;
    color: #FFF;
  }
  .header__auth-button{
    background: #FFF;
    border: none;
    border-radius: 50px;
    padding: 5px 10px;
    color: #009CE5;
  }
  .chat-area{
    background-color: #FFF;
    border-radius: 50px;
    max-width: 500px;
    width: 90%;
    box-sizing: border-box;
    padding: 25px;
    margin: 20px 5% 0;
  }
  .conversation{
    width: 100%;
    height: calc(100vh - 200px);
    margin-bottom: 10px;
    overflow: auto;
    display: grid;
    grid-template-rows: 1fr;
  }
  .conversation__inner{
    padding: 0;
    margin: 0;
    width: 100%;
    grid-row-start: 2;
  }
  .conversation__inner > li{
    list-style: none;
  }
  .conversation__inner > li:nth-child(n+2){
    margin-top: 20px;
  }
  .conversation__user-image{
    position: relative;
    width: 40px;
    height: 40px;
    overflow: hidden;
    border-radius: 50%;
    float: left;
  }
  .conversation__user-image img{
    width: 100%;
    height: auto;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
  }
  .conversation__col{
    margin-left: 50px;
  }
  .conversation__user-name{
    margin: 0 0 5px;
    color: #878787;
  }
  .post{
    width: 100%;
    display: flex;
    align-items: center;
  }
  .post input{
    width: 80%;
    height: 35px;
  }
  .post button{
    width: 20%;
    height: 35px;
    background: #FFF;
    border: 1px solid #878787;
    color: #878787;
  }
</style>
