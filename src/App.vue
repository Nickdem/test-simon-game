<template>
  <div class="home">
    <div class="container">
      <h1>Lvl - {{level}}</h1>
      <div class="playfield">
        <Btn v-for="btn of gameBtn" :key="btn" :text="btn" @handler="click" class="playfield-button" :ref="'btn' + btn" />
      </div> 
      <div class="game-over" v-if="state == 'фэйл'">
        <GameOver @re="restart"/>
      </div>
      <div class="button-group" v-if="level === 0">
        <Btn v-for="btn of menuBtn" :key="btn" :text="btn" @handler="start" />
      </div>
    </div>
    <!-- <div ref="audio"></div>  build -->
  </div>
</template>

<script>
import Btn from './components/Btn'
import GameOver from './components/GameOver'
export default {
  name: 'App',
  data: () => ({
    menuBtn: ['легкий', 'средний', 'сложный'],
    gameBtn: [0, 1, 2, 3],
    level: 0,
    timeout: 1500,
    state: 'жду',
    currentSequence: []
  }),
  components: {Btn, GameOver},
  methods: {
    start (value) {
      this.state = 'продолжаю'

      if (value === 'сложный') {
        this.timeout = 400
      } else if (value === 'средний') {
        this.timeout = 1000
      } else { 
        this.timeout = 1500
      }

      this.levelUp()
    },

    play (sequence = []) {     
      sequence.forEach((n, i) => {
        this.state = 'жду'

        setTimeout(() => {
          const key = `btn${n}`
          let btn = this.$refs[key][0].$el
          btn.click(n)

          if (i == sequence.length - 1) {
            this.state = 'продолжаю'
          } 
        }, this.timeout * i)
      })
    },

    levelUp () {
      this.level++
      this.currentSequence = []

      for (let i=0; i<this.level; i++) {
        this.currentSequence.push(this.randomNumber(0,3))
      }
      
      this.play(this.currentSequence)
    },

    randomNumber (min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min
    },

    click (n) {
      const key = `btn${n}`
      let btn = this.$refs[key][0].$el
      btn.classList.add('click')
      
      setTimeout(() => {
          btn.classList.remove('click') 
      }, 200)
      //dev
      this.playAudio(require(`./sounds/${n + 1}.ogg`))  
      //this.playAudio(n + 1) // build
      this.state == 'продолжаю' && this.checkSequence(n)
    },

    playAudio (sound) {
      //dev
      const audio = new Audio(sound)
      audio.play()

      //build - добавить папку sound
      // const audio = this.$refs.audio
      // audio.innerHTML = `<audio autoplay><source src="./sounds/${sound}.ogg" type="audio/ogg" /></audio>`
    }, 
       
    checkSequence (key) {
      const expected = this.currentSequence.shift()
      
      if (expected == key) {
        if (!this.currentSequence.length) {
          setTimeout(() => {
            this.state = 'жду'
            this.levelUp()
          }, 1000)
        }
      } else {
        this.gameOver()
      }
    },

    gameOver () {
      this.state = 'фэйл'
    },

    restart () {
      this.timeout = 1500
      this.state = 'жду'
      this.level = 0
      this.currentSequence = []
    }
  }
}
</script>

<style>
body {
  margin: 0;
}

.home {
  background: #868686;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  width: 1000px;
  height: 700px;
  background: #ffffff;
  display: flex;
  flex-direction: column-reverse;
  justify-content: center;
  align-items: center;
}

.playfield {
  width: 508px;
  margin: 1rem auto;
}

.playfield-button {
  outline: none;
  cursor: pointer;
  width: 250px;
  height: 250px;
  margin: 0;
  border: none;
  opacity: .3;  
}

.playfield-button:first-child {
  border-top-left-radius: 100%;
  background-color: lightblue;
}

.playfield-button:nth-child(2) {
  border-top-right-radius: 100%;
  background-color: red;
}

.playfield-button:nth-child(3) {
  border-bottom-left-radius: 100%;
  background-color: yellow;
}

.playfield-button:last-child {
  border-bottom-right-radius: 100%;
  background-color: green;
}

.click {
  opacity: 1;
  pointer-events: none;
  box-shadow: 0 0  15px black;
}

.game-over {
  position: absolute;
  background: #ffffff;
  opacity: .8;
  width: 700px;
  height: 700px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.button-group {
  width: 300px;
  display: flex;
  justify-content: space-between;
}

.button-group button {
  background: none;
  border: none;
  font-size: 18px;
}

.button-group button:first-child {
  color: green
}

.button-group button:nth-child(2) {
  color: orange
}

.button-group button:last-child {
  color: red
}
</style>
