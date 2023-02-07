<script>
  import tmi from 'tmi.js';

  export default {
    name: 'App',
    data() {
      return {
        users: [],
        usersLeftColumn: [],
        usersRightColumn: [],
        leaderboard: [],
        round: 1,
        timerCount: 20,
        timerEnabled: false,
        life: 3,
      }
    },
    mounted() {
      this.connect();
    },
    watch: {
      timerEnabled(value) {
        if (value) {
          setTimeout(() => {
            this.timerCount--;
          }, 1000);
        }
      },
      timerCount: {
        handler(value) {
          if (value > 0 && this.timerEnabled) {
            setTimeout(() => {
              this.timerCount--;
            }, 1000);
          } else if (value === 0) {
            const mostPopularColumn = this.usersLeftColumn.length > this.usersRightColumn.length ? this.usersLeftColumn : this.usersRightColumn;
            const leastPopularColumn = this.usersLeftColumn.length < this.usersRightColumn.length ? this.usersLeftColumn : this.usersRightColumn;
            if (this.usersLeftColumn.length === this.usersRightColumn.length && mostPopularColumn.length > 0) {
              this.leaderboard.forEach(user => {
                if (user.life > 0) user.life--;
              });
            } else if (!(mostPopularColumn.length === this.leaderboard.length) && mostPopularColumn.length > 0) {
              this.leaderboard.forEach(user => {
                if (!leastPopularColumn.some(userPopular => userPopular.id === user.id) && user.life > 0) {
                  user.life--;
                }
              });
            }
            this.leaderboard = this.leaderboard.sort((a, b) => b.life - a.life);
            this.usersLeftColumn = [];
            this.usersRightColumn = [];
            this.timerCount = 20;
            this.timerEnabled = false;
            this.round++;
          }
        },
        immediate: true
      }
    }, 
    methods: {
      startGame() {
        this.timerEnabled = true;
        this.leaderboard = this.leaderboard.length === 0 ? this.users : this.leaderboard;
      },
      connect() {
        const client = new tmi.Client({
          channels: [ 'plotsdechantier' ]
        });

        client.connect();

        client.on('message', (channel, tags, message, self) => {
          if (!this.timerEnabled) {
            if (message === '!play' && !this.users.some(user => user.id === tags['user-id'])) {
              this.users.push({
                id: tags['user-id'],
                name: tags['display-name'],
                life: this.life,
              });
            }
          } else {
            const userCurrent = this.users.find(user => user.id === tags['user-id']);
            message = message.toLowerCase();
            if (userCurrent && userCurrent.life > 0 ) {
              if ((message === '!g' || message === '!gauche') && this.timerCount > 0) {
                if (!this.usersLeftColumn.some(user => user.id === userCurrent.id)) {
                  this.usersLeftColumn.push(userCurrent);         
                }
                if (this.usersRightColumn.some(user => user.id === userCurrent.id)) {
                  this.usersRightColumn = this.usersRightColumn.filter(user => user.id !== tags['user-id']);
                }
              }
              if ((message === '!d' || message === '!droite' && this.timerCount > 0)) {
                if (!this.usersRightColumn.some(user => user.id === userCurrent.id)) {
                  this.usersRightColumn.push(userCurrent);
                }
                if (this.usersLeftColumn.some(user => user.id === userCurrent.id)) {
                  this.usersLeftColumn = this.usersLeftColumn.filter(user => user.id !== tags['user-id']);
                }
              }
            }
          }
        });
      }
    }
  }
</script>

<template>
  <main class="container">
    <div class="game">
      <h1 class="timer">Temps restant : {{ timerCount }}s</h1>
      <h1 class="round">Manche n°{{ round }}</h1>
      <div class="game-columns">
        <div class="left-column">
          <h2>Gauche</h2>
          <div v-for="user in usersLeftColumn" :key="user.id">
            {{ user.name }}
          </div>
        </div>
        <div class="right-column">
          <h2>Droite</h2>
          <div v-for="user in usersRightColumn" :key="user.id">
            {{ user.name }}
          </div>
        </div>
      </div>
      <button class="button" @click="startGame">Start</button>
      <h2>Pour rejoindre !play</h2>
      <h2>Pour jouer !g ou !gauche ou !d ou !droite</h2>
      <div class="leaderboard">
        <h2>Vies restantes</h2>
        <!-- leaderboard that print only 10 first -->
        <div v-for="user in leaderboard.slice(0, 5)" :key="user.id">
          <p :class="user.life === 0 ? 'test' : 'test2'">{{ user.name }} : {{ user.life }}</p>
        </div>
      </div>
      <div class="users">
        <h2>Joueurs</h2>
        <div v-for="user in users" :key="user.id">
          {{ user.name }}
        </div>
      </div>
    </div>
  </main>
</template>

<style>
.test {
  color: red;
  text-decoration: line-through;
}
.test2 {
  color: green;
}
.container {
  width: 90vw;
  height: 90vh;
  display: flex;
  flex-direction: column;
}
.game {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.timer {
  margin-bottom: 50px;
}
.game-columns {
  display: flex;
  flex-direction: row;
}
.left-column {
  margin-right: 50px;
}
.right-column {
  margin-left: 50px;
}
.leaderboard {
  position: absolute;
  left: 0;
  top: 0;
}
.users {
  position: absolute;
  right: 0;
  top: 0;
}
.button {
  margin: 50px 0;
  width: 100px;
  height: 50px;
  background-color: green;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
