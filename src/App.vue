<template>
  <ExpTracker/>
</template>

<script>
import ExpTracker from './components/ExpTracker.vue'

export default {
  name: 'App',
  components: {
    ExpTracker
  }
}
</script>

<style>

*{
margin: 0;
}

#app {
  background-color: #383838;
  height:100vh;
}
</style>
