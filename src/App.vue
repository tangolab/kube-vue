<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/cluster">Cluster</router-link> |
      <router-link to="/nodes">Nodes</router-link> |
      <router-link to="/namespaces"> Namespaces</router-link> 
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
a, a:visited{
  color:white;
}
body {
  margin: 30px;
  background-color: black;
  font-family: "Arial";
  color: white;
}

</style>
