<template>
  <div class="auth-widget purple-gradient" id="login">
    <h2>Login</h2>
    <p>Email:</p>
    <input v-model="email" type="text" class="auth-textbox"
          @keyup.enter="signIn()"/>
    <p>Password:</p>
    <input v-model="password" type="password" class="auth-textbox"
          @keyup.enter="signIn()"/> <br><br>
    <button @click="signIn()" class="material-button-large ">Login!</button> <br><br>

    <p class="err-text">{{err}}</p>

    <p>Don't have an account yet?
      <router-link to="/register" tag="span" class="inline-link">
        Register!
      </router-link>
    </p>

  </div>
</template>

<script>
import * as firebase from 'firebase';

export default {
    data() {
        return {
            email: "",
            password: "",
          err: ''
        }
    },
    methods: {
        signIn() {
            firebase.auth().signInWithEmailAndPassword(this.email, this.password)
            .then((user) => {
                console.log(user);
              this.$router.push('/');
            }).catch((error) => {
                this.err = error.message;
            });
        }
    }
 }
</script>

<style scoped lang="scss">
@import '@/GlobalVars.scss';

  #login {

  }

/*  Note that this only applies to this page's material button: */
  .material-button-large {
    background: $gray;
    color: white;
  }
</style>
