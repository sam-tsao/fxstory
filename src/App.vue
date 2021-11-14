<template>
  <div id="app">
    <h1>FXSTORY</h1>
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
}
h1 {
  padding: 0 2vw;
}

footer {
  padding-top: 5vh;
  text-align: right;
}
</style>
