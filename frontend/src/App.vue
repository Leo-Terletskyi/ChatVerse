<script>
import axios from "axios";
import {toast} from "bulma-toast";

export default {
  data() {
    return {
      isMenuOpen: false
    }
  },
  beforeCreate() {
    this.$store.commit('initializeStore')
    const token = this.$store.state.token
    if (token) {
      axios.defaults.headers.common['Authorization'] = "Token " + token
    } else {
      axios.defaults.headers.common['Authorization'] = ""
    }
  },
  created() {
    if (this.$store.state.token) {
      this.setUserOnline(true)
      window.onbeforeunload = (e) => {
        // e.preventDefault()
        this.setUserOnline(false)
      }
    }
  },
  methods: {
    toggleMenu() {
      this.isMenuOpen = !this.isMenuOpen
    },
    async setUserOnline(isOnline = false) {
      try {
        await axios.patch(`http://127.0.0.1:8000/api/v1/account/${this.$store.state.username}/update/`, {is_online: isOnline})
            .then(() => {
            })
      } catch (error) {
        console.error('Failed to update online status:', error);
      }
    },
    async logOut() {
      await this.setUserOnline(false)

      await axios.post('http://127.0.0.1:8000/api/v1/auth/token/logout/')
          .then(() => {
            this.$store.commit('removeUsername')
            this.$store.commit('removeToken')

            toast({
              message: 'You have successfully logged out',
              type: 'is-success',
              dismissible: true,
              pauseOnHover: true,
              duration: 1500,
              position: 'bottom-right'
            })

            setTimeout(() => {
              this.$router.push('logIn')
            }, 500)
          })
          .catch((err) => {
            console.log(err)
          })
    }
  }
}
</script>

<template>
  <nav class="navbar" role="navigation" aria-label="main navigation">
    <div class="navbar-brand mr-5">
      <router-link to="/" class="navbar-item">
        <img src="./assets/logo-no-background.svg" alt="" height="100">
      </router-link>
      <a role="button" class="navbar-burger" aria-label="menu" aria-expanded="false" data-target="navbarBasicExample" @click="toggleMenu">
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
      </a>
    </div>

    <div id="navbarBasicExample" class="navbar-menu" :class="{ 'is-active': isMenuOpen }">
      <div class="navbar-start">
        <router-link to="/profile" class="navbar-item mr-3">
          <span class="icon is-small">
            <font-awesome-icon icon="user"/>
          </span>
          <span class="ml-2">Profile</span>
        </router-link>

        <router-link to="/contact-management" class="navbar-item mr-3">
          <span class="icon is-small">
            <font-awesome-icon icon="fa-solid fa-address-book"/>
          </span>
          <span class="ml-2">Contact management</span>
        </router-link>

        <div class="navbar-item">
          <form action="/user-search" method="get">
            <div class="field has-addons">
              <div class="control">
                <input class="input" type="text" placeholder="User search" name="query">
              </div>
              <div class="control">
                <button class="button has-background-orange" type="submit">
                  <font-awesome-icon icon="search"/>
                </button>
              </div>
            </div>
          </form>
        </div>
      </div>

      <div class="navbar-end">
        <div class="navbar-item pr-0 mr-2">
          <div class="buttons">
            <template v-if="!this.$store.state.isAuthenticated">
              <router-link to="/sign-up" class="btn-grad-lightgreen mt-0">
                <strong>Sign up</strong>
              </router-link>
              <router-link to="/login" class="btn-grad-lightblue mt-0 mr-0">
                Log in
              </router-link>
            </template>

            <template v-else>
              <button class="btn-grad-rude auth-form-btn-style mt-0 mx-0" @click="logOut">Logout</button>
            </template>
          </div>
        </div>
      </div>
    </div>
  </nav>

  <div class="main-content">
    <router-view></router-view>
  </div>

  <div class="main-footer has-text-centered">
    <p class="pt-1">
      <a href="https://github.com/Leo-Terletskyi"> &#169; Leonid Terletskyi (2023)</a>
    </p>
  </div>

</template>

<style scoped>

</style>
