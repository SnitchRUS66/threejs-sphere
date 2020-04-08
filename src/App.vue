<template>
  <div id="app">
    <!-- <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div> -->
    <router-view/>
  </div>
</template>

<style lang="stylus">
* {
    box-sizing: border-box;
}

html,
body {
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
}

#app {
    font-family: 'Roboto Condensed', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    height: 100%;
}
</style>
