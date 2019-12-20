<template>
    <div id="app" class="hello">
  <h1>Welcome to the Serverless Voting App. Now with Amplify!</h1>
    <h2>Sign In</h2>
    <div class='auth'>
        <amplify-authenticator></amplify-authenticator>
    </div>
        <h4>You can vote as many times as you like. Click away!</h4>
  <b-row align-h="center" class="mt-5">
        <b-card-group deck>
          <b-card bg-variant="success" text-variant="white" header="Vote Yes" class="text-center" footer-tag="footer">
            <b-card-text>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</b-card-text>
            <b-button size="lg" variant="primary"  @click="vote('yes')">Button</b-button>
            <em slot="footer">{{ votesYes }} voted</em>
          </b-card>
  <b-card bg-variant="danger" text-variant="white" header="Vote No" class="text-center" footer-tag="footer">
            <b-card-text>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</b-card-text>
            <b-button size="lg" variant="primary" @click="vote('no')">Button</b-button>
            <em slot="footer">{{ votesNo }} voted</em>
          </b-card>
        </b-card-group>
      </b-row>
      <b-row align-h="center" class="mt-5">
        <p>Questions? Ask James <a href="https://twitter.com/jbesw">@jbesw</a>.</p>
      </b-row>
      <amplify-sign-out></amplify-sign-out>
    </div>  
  </template>

  <script>
import { API } from 'aws-amplify';
import { AmplifyEventBus } from 'aws-amplify-vue';
import { Auth } from 'aws-amplify';

export default {
  name: 'app',
  data() {
    return {
      signedIn: false,
      apiName: 'tanjapollAPI',
      votesYes: 0,
      votesNo: 0
    }
  },
  methods: {
    vote: async function (vote) {
      const init = {
        queryStringParameters: {
          vote
        }
      }
      const response = await API.post(this.apiName, '/votes', init)
      if (vote === 'yes') this.votesYes = response.data.Attributes.votesYes
      if (vote === 'no') this.votesNo = response.data.Attributes.votesNo
    },
    updateVotes: async function () {
      const response = await API.get(this.apiName, '/votes/poll-001')
      this.votesNo = response[0].votesNo
      this.votesYes = response[0].votesYes    
    },
    async isUserSignedIn(){
      try {
        const userObj = await Auth.currentAuthenticatedUser();
        this.signedIn = true;
        console.log("MD: [isUserSignedIn] Current user:" + JSON.stringify(userObj));
      } catch (error) {
        this.signedIn = false;
    }
  },
  created () {
      this.updateVotes()
      setInterval(this.updateVotes, 9999999999) 
      this.isUserSignedIn();
      AmplifyEventBus.$on('authState', info => {
        if (info === "signedIn") {
          this.signedIn = true;
        } else {
          this.signedIn = false;
        }
      })
  }
}
}
</script>

<style>
    #app {
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 60px;
    }
    </style>