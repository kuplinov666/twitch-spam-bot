<template>
  <div id="app">
    <header>
      <h1>Twitch Spam Bot</h1>
    </header>
    <main>
      <div class="wrapper inputs console">
        <h2>Console</h2>
        <hr class="green">
        <div class="item">
          Channel:<input :disabled="isStarted" type="text" v-model.lazy.trim="channel">
        </div>
        <div class="start">
          <button @click="Start">
            {{isStarted? 'Stop' : 'Start'}}
          </button>
        </div>
      </div>
      <div class="wrapper inputs">
        <h2>Accounts <a target="_blank" href="http://twitchapps.com/tmi/">(Get Token)</a></h2>
        <hr class="red">
        <div class="item accounts" v-for="(item, index) in accounts" :key="index">
          <input :disabled="isStarted" type="text" placeholder="Username" v-model.trim="item.name">
          <input :disabled="isStarted" type="text" placeholder="oauth:******************************" v-model.trim="item.token">
          <button :disabled="isStarted" v-if="accounts.length - 1" @click="accounts.splice(index, 1)">
            <img src="@/assets/trash.svg">
          </button>
        </div>
        <div class="item">
          <button :disabled="isStarted" @click="accounts.push({name: '', token: ''})">
            <img src="@/assets/plus.svg">
          </button>
        </div>
      </div>
      <div class="wrapper inputs">
        <h2>Messages</h2>
        <hr class="blue">
        <div class="item messages" v-for="(item, index) in messages" :key="index">
          <input :disabled="isStarted" type="text" placeholder="Message" v-model.trim="item.message">
          <button :disabled="isStarted" v-if="messages.length - 1" @click="messages.splice(index, 1)">
            <img src="@/assets/trash.svg" width="20">
          </button>
        </div>
        <div class="item">
          <button :disabled="isStarted" @click="messages.push({message: ''})">
            <img src="@/assets/plus.svg">
          </button>
        </div>
      </div>
      <div class="wrapper stream">
        <iframe id="stream" :src="`https://player.twitch.tv/?muted=true&parent=${iframeParent}&channel=${channel}`" width="100%" frameborder="0"></iframe>
        <iframe id="chat" :src="`https://www.twitch.tv/embed/${channel}/chat?parent=${iframeParent}`" width="100%" frameborder="0"></iframe>
      </div>
    </main>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      isStarted: false,
      channel: "",
      accounts: [],
      messages: [],
      msgPerMin: 5,
      threads: 1,
      bots: []
    }
  },
  computed: {
    iframeParent(){
      return window.location.hostname
    }
  },
  created(){
    this.accounts.push({name: "", token: ""})
    this.messages.push({message: ""})
    const data = localStorage.getItem('data')
    if(data !== null)
      Object.assign(this.$data, JSON.parse(data))
    this.isStarted = false
    this.bots = []
  },
  methods:{
    Start(){
      localStorage.setItem('data', JSON.stringify(this.$data))
      this.isStarted = !this.isStarted
      if(this.isStarted)
        this.accounts.forEach(e => {
          const bot = new Worker("bot.js")
          bot.postMessage({
            account: e,
            messages: this.messages,
            channel: this.channel,
          })
          this.bots.push(bot)
        })
      else{
        this.bots.forEach(bot => bot.terminate())
        this.bots = []
      }
    },
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Play:wght@400;700&display=swap');

*{
  color: white;
  margin: 0;
  padding: 0;
  outline: none;
  font-family: 'Play', sans-serif;
}

#app{
  width: 100vw;
  min-width: 1600px;
  height: 100vh;
  min-height: 900px;
  background:#0b0f13;
  font-size: 16px;
}

hr{
  height: 2px;
  margin-bottom: 20px;
}

img{
  width: 20px;
}

.red{
  $color: red;
  background: $color;
  border-color: $color;
}

.blue{
  $color: #00BFFF;
  background: $color;
  border-color: $color;
}

.green{
  $color: rgb(64, 128, 82);
  background: $color;
  border-color: $color;
}

h1, h2{
  margin-left: .6rem;
}

h2{
  margin-bottom: .65rem;
  a{
    font-size: 1.3rem;
  }
}

$header-height: 60px;
header{
  $height: 60px;
  padding-left: 20px;
  height: $header-height;
  
  h1{
    line-height: $header-height;
    font-weight: 600;
    font-size: 2rem;
    letter-spacing: 2px;
  }
}

input, button{
  background: #171c23;
  border: transparent;
  border-radius: 3px;
  border: 1px solid rgba(0, 0, 0, 0.5);
  box-shadow: 0 1px 0 1px inset rgba(0,0,0,.3), 0 1px 1px 0 rgba(255,255,255,.04);
  font-weight: 600;
  padding: .5rem;
  margin: 0 .5rem;
}

button{
  background: transparent;
  border: transparent;
  cursor: pointer;
  box-shadow: none;
}

.item{
  display: grid;
  align-items: center;
  height: 35px;
  margin-bottom: 10px;
}
main{
  display: grid;
  grid-template-columns: repeat(4, 1fr);

  .wrapper{
    $margin: 8px;
    margin: 0 $margin;
    height: calc(100vh - #{$header-height} - #{$margin});
    box-sizing: border-box;
    
    &.inputs{
      padding: 8px;
      border: 2px solid #1b1b1b;
      background-color:#1a1f28;

      &.console{
        .item{
          grid-template-columns: 110px 1fr;
        }

        .start{
          margin-top: 20px;
          text-align: center;

          button{
            padding: 0.5rem 2rem;
            background: #ffffff;
            color:#0b0f13;
            font-size: 1.5rem;
          }
        }
      }

      .item{
        &.accounts{
          grid-template-columns: 1fr 1fr 50px;
        }

        &.messages{
          grid-template-columns: 1fr 50px;
        }
      }
    }

    &.stream{
      display: flex;
      flex-direction: column;

      #stream{
        flex-grow: 1;
        margin-bottom: $margin * 2;
      }

      #chat{
        flex-grow: 6;
      }
    }
  }
}
</style>
