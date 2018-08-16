<template>
  <form class="form" @submit.prevent="login">
    <the-header v-once />
    <div class="form-group">
      <label for="email">Login name</label>
      <div class="errors" v-if="errorMessage">
        {{ errorMessage }}
      </div>
      <input
        id="email"
        type="email"
        autocomplete="username"
        placeholder="email"
        required
        v-model="username"
      >
    </div>
    <div class="form-group">
      <label for="password">Password</label>
      <input
        type="password"
        id="password"
        placeholder="email"
        v-model="password"
        autocomplete="current-password"
        required
      >
    </div>
    <button type="submit">{{ isWorking ? 'Working ...' : 'LOGIN'}}</button>
  </form>

</template>

<script>
import axios from 'axios'
import moment from 'moment'
import TheHeader from '@/components/TheHeader'

export default {
  name: 'LoginForm',
  components: { TheHeader },
  data: () => ({
    username: '',
    password: '',
    errorMessage: '',
    isWorking: false
  }),
  methods: {
    login () {
      // validation
      this.errorMessage = ''
      this.isWorking = true
      axios.post(
        'https://vue-todos.robertmckenney.ca/oauth/token',
        {
          grant_type: 'password',
          client_id: 15,
          client_secret: 're9k0iuBjkFf4qSvlGMZOjrihq8r9VreRknb4sR6',
          username: this.username,
          password: this.password,
          scope: '*'
        }
      ).then(response => {
        const apiTokens = {
          expires_at: moment().add(response.data.expires_in, 'second').format(),
          ...response.data
        }
        this.$emit('userDidAuthenticate', apiTokens)
        this.username = ''
        this.password = ''
      }).catch(error => { this.handleError(error) })
        .finally(() => { this.isWorking = false })
      // redirect to the main application view
    },
    handleError (error) {
      if (error.response) {
        switch (error.response.status) {
          case 400: {
            this.errorMessage = 'Both username and password are required.'
            break
          }
          case 401: {
            this.errorMessage = 'Incorrect username or password.'
            break
          }
          default: {
            this.errorMessage = 'Sorry. There was an problem on the server.'
          }
        }
      } else if (error.request) {
        console.log(error.request)
      } else {
        console.log('Error: ', error.message)
      }
      console.log(error.config)
    }
  }
}
</script>

<style>
.form {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    padding: 1.5em;
}
.form-group {
    margin: 1em 0;
    display: flex;
    flex-direction: column;
}
.form-group label {
    margin-bottom: 0.25em;
    font-size: 0.875em;
    color: hsl(0, 0%, 22%);
}
.form-group input {
    border-radius: 0.25rem;
    border: 1px solid hsl(0, 0%, 88%);
    padding: 0.25em;
    font-size: 1.1em;
}
.form-group input::placeholder {
    opacity: 0.54;
}
.form button {
    background: #239CCC;
    border: 1px solid #239CCC;
    border-radius: 0.25rem;
    padding: 0.5rem 0.5rem;
    color: hsl(0, 0%, 99%);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-weight: 500;
    font-size: 1.1rem;
    width: 100%;
    box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.25);
    outline: none;

    &:active {
      box-shadow: none;
      background: #239CCC;
      border: 1px solid #239CCC;
    }
  }
.errors {
    border-radius: 0.25em;
    border: 1px solid hsl(348, 83%, 93%);
    color: hsl(348, 83%, 47%);
    background-color: hsl(348, 63%, 97%);
    padding: 1.5em;
}
</style>
