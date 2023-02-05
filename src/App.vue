<script>
  //import HelloWorld from './components/HelloWorld.vue'
  //import TheWelcome from './components/TheWelcome.vue'
  import tmi from 'tmi.js';

  export default {
    name: 'App',
    components: {
      //HelloWorld,
      //TheWelcome
    },
    data() {
      return {
        users: [],
        started: false
      }
    },
    mounted() {
      this.connect();
    },
    methods: {
      connect() {
        const client = new tmi.Client({
          channels: [ 'plotsdechantier' ]
        });

        client.connect();

        client.on('message', (channel, tags, message, self) => {
          if (!this.started) {
            if (message === '!play' && !this.users.some(user => user.id === tags['user-id'])) {
              this.users.push({
                id: tags['user-id'],
                name: tags['display-name'],
                score: 0
              });
            }
          } else {
            const user = this.users.find(user => user.id === tags['user-id']);
            if (user) {
              user.score += 1;
            }
          }
        });
      }
    }
  }
</script>

<template>
  <header>
    <!--<img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>-->
  </header>

  <main>
    <div v-for="user in users">
      <p>{{ user.name }} 
        <span v-if="started">
          : {{ user.score }}
        </span>
      </p>

    </div>
    <button @click="started = true">Start</button>
    <!--<TheWelcome />-->
  </main>
</template>

<style scoped>
/*header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}*/
</style>
