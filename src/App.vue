<template>
  <div class="home">
    <div class="container">
      <h1>Lvl - {{level}}</h1>
    <div class="playfield">
      <button v-for="num of gameBtn" :key="num" type="button" :ref="'btn'+num" class="playfield-button" @click='click(num)'></button>
    </div> 
    <div class="game-over" v-if="state == 'фэйл'">
      <GameOver @re="restart"/>
    </div>
    <div class="button-group" v-if="level === 0">
      <MenuBtn v-for="btn of menuBtn" :key="btn" :text="btn" @handler="start" />
    </div>
    </div>
  </div>
</template>

<script>
import MenuBtn from './components/MenuBtn'
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
  components: {MenuBtn, GameOver},
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
          let lb = this.$refs[key][0]
          lb.click(n)

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
      let lb = this.$refs[key][0]
      lb.classList.add('click')
      
      setTimeout(() => {
          lb.classList.remove('click') 
      }, 200)

      this.playSound(require(`./sounds/${n + 1}.ogg`))
      
      this.state == 'продолжаю' && this.checkSequence(key)
    },

    playSound (sound) {
      const audio = new Audio(sound)
      audio.play()
    }, 
       
    checkSequence (key) {
      const keymap = {"btn0": 0, "btn1": 1, "btn2": 2, "btn3": 3}
      const keyindex = keymap[key]
      const expected = this.shiftSequence()

      if (expected == keyindex) {
        if (!this.sequence().length) {
          setTimeout(() => {
            this.state = 'жду'
            this.levelUp()
          }, 1000)
        }
      } else {
        this.gameOver()
      }
    },

    sequence () {
      return this.currentSequence
    },

    shiftSequence () {
      return this.currentSequence.shift()
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
  margin: 0 1px;
  border: none;
  opacity: .3;  
}

.playfield-button:first-child {
  border-top-left-radius: 100%;
  background-color: blue;
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
  width: 500px;
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
