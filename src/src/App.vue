<template>
  <div id="app">
    <nav class="navbar is-transparent is-info">
      <div class="navbar-brand">
        <router-link class="navbar-item" to="/">
          <img src="/img/icons/favicon-32x32.png" alt="Luke-Not-Luke" width="30" height="30">
        </router-link>
        <router-link class="navbar-item" to="/">Home</router-link>
        <router-link class="navbar-item" to="/about">About</router-link>
      </div>
    </nav>
    <router-view/>
  </div>
</template>

<style lang="stylus">
@import '../node_modules/bulma/css/bulma.css'

#app
  font-family 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #2c3e50

</style>
