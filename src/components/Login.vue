<template>
  <div class="q-ma-md">
    <q-tabs v-model="selectedTab" color="primary" active-bg-color="primary" class="bg-secondary text-white" align="justify">
      <q-tab name="tab-1" icon="fas fa-unlock" label="Login"/>
      <q-tab name="tab-2" icon="fas fa-hand-point-up" label="Register"/>
    </q-tabs>
    <q-tab-panels v-model="selectedTab">
      <q-tab-panel class="no-border" name="tab-1">
        <div class="alert alert-danger" v-if="error">
          <p>There was an error, unable to sign in with those credentials.</p>
        </div>
        <form autocomplete="off" @submit.prevent="login" method="post">
          <div class="q-pa-sm">
            <q-input label="Email" type="email" v-model="email" required />
          </div>
          <div class="q-pa-sm">
            <q-input label="Password" type="password" v-model="password" required />
          </div>
          <div class="q-pa-sm text-center">
            <q-btn @click="login">Sign in</q-btn>
          </div>
        </form>
      </q-tab-panel>
      <q-tab-panel class="no-border" name="tab-2">
        Your phone number has already been confirmed. Please now choose a password and confirm your email address to provide an extra layer of security. Once registered and authorised by your administrator, you will be able to edit confidential member details.
        <form autocomplete="off" @submit.prevent="login" method="post">
          <div class="q-pa-sm">
            <q-input label="Name" v-model="newname" required />
          </div>
          <div class="q-pa-sm">
            <q-input label="Email" type="email" v-model="newemail" required />
          </div>
          <div class="q-pa-sm">
            <q-input label="Cellphone" v-model="newphone" required readonly />
          </div>
          <div class="q-pa-sm">
            <q-input label="Password" type="password" v-model="newpassword" required />
          </div>
          <div class="q-pa-sm text-center">
            <q-btn @click="register">Register</q-btn>
          </div>
        </form>
      </q-tab-panel>
    </q-tab-panels>
  </div>
</template>

<script>
export default {
  data () {
    return {
      email: '',
      selectedTab: 'tab-1',
      password: '',
      newpassword: '',
      newemail: '',
      newname: '',
      newphone: localStorage.getItem('CHURCHNET_verifiedphone'),
      error: 0,
      verified: false
    }
  },
  mounted () {
    if ((!localStorage.getItem('CHURCHNET_phonetoken')) && (!localStorage.getItem('CHURCHNET_verifiedphone'))) {
      this.verification()
    }
    this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
    this.$axios.post(process.env.API + '/checkphone',
      {
        phone: this.newphone
      })
      .then(response => {
        this.newname = response.data.firstname + ' ' + response.data.surname
        this.newemail = response.data.email
      })
      .catch(function (error) {
        console.log(error)
      })
  },
  methods: {
    login () {
    },
    verification () {

    },
    register () {
      this.$axios.post(process.env.API + '/register',
        {
          email: this.newemail,
          password: this.newpassword,
          name: this.newname,
          cellphone: this.newphone
        })
        .then(response => {
          this.email = this.newemail
          this.password = this.newpassword
          this.login()
        })
        .catch(function (error) {
          this.error = error
        })
    }
  }
}
</script>

<style>
</style>
