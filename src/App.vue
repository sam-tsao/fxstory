<template>
  <div id="app">
    <div>FXSTORY</div>
    <router-view/>
    <footer>
      made by <a href="https://twitter.com/sam___tsao" target="_blank">Sam Tsao </a>
    </footer>
  </div>
</template>

<style>
* {
  font-family: Courier, monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

footer {
  padding-top: 5vh;
  text-align: right;
}
</style>
