<template>
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
        <p class="error-message" v-if="errorMessage">{{ errorMessage }}</p>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  name: 'Chat',
  props: [
    'isSignedIn',
    'userName',
    'userPic'
  ],
  data () {
    return {
      message: null,
      errorMessage: null,
      messageList: []
    }
  },
  created () {
    this.loadMessages()
  },
  methods: {
    postMessage () {
      let that = this
      if (!that.isSignedIn || !that.message) return
      firebase.database().ref('/messages/').push({
        name: that.userName,
        text: that.message,
        profilePicUrl: that.userPic
      })
      .then( data => {
        that.errorMessage = null
        that.message = null
      })
      .catch( error => {
        that.errorMessage = '正しく入力してください'
      })
    },
    loadMessages () {
      firebase.database().ref('/messages/').on('value', (snapshot) => {
        if (snapshot) {
          let rootList = snapshot.val()
          let messageList = []
          Object.keys(rootList).forEach((val, key) => {
            rootList[val].id = val
            messageList.push(rootList[val])
          })
          this.messageList = messageList
        }
      })
    }
  }
}
</script>

<style scoped lang="scss">
  .chat-area {
    background-color: #FFF;
    border-radius: 50px;
    max-width: 500px;
    width: 90%;
    box-sizing: border-box;
    padding: 25px 35px;
    margin: 20px 5% 0;
  }

  .conversation {
    width: 100%;
    height: calc(100vh - 200px);
    margin-bottom: 10px;
    overflow: auto;
    display: grid;
    grid-template-rows: 1fr;

    &__inner {
      padding: 0;
      margin: 0;
      width: 100%;
      grid-row-start: 2;
      > li {
        list-style: none;
        &:nth-child(n+2) {
          margin-top: 20px;
        }
      }
    }

    &__user-image {
      position: relative;
      width: 40px;
      height: 40px;
      overflow: hidden;
      border-radius: 50%;
      float: left;
      img {
        width: 100%;
        height: auto;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translateX(-50%) translateY(-50%);
      }
    }

    &__col {
      margin-left: 50px;
    }

    &__user-name {
      margin: 0 0 5px;
      color: #878787;
    }
  }

  .post {
    width: 100%;
    display: flex;
    align-items: center;
    position: relative;
    input {
      width: 80%;
      height: 35px;
    }
    button {
      width: 20%;
      height: 35px;
      background: #FFF;
      border: 1px solid #878787;
      color: #878787;
    }
  }

  .error-message {
    color: #f00;
    font-size: .8em;
    margin: 5px 0 0;
    position: absolute;
    left: 0;
    bottom: 0;
    transform: translateY(100%);
  }
</style>
