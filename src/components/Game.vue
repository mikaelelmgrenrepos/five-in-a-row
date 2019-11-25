<template>
  <div class="">
    <div v-if="gameOn">
      <b-spinner type="grow" label="Spinning"></b-spinner>
    </div>
    <div v-else>
      <h1>{{ gameOn }}</h1>
      <h3>The Game</h3>
    </div>
    
  </div>
</template>

<script lang="ts">
export default {
  name: "Game",
  props: ["gameOn"],

  data() {
    return {
      isLoading: true,
      playerOne: "",
      playertwo: ""
    };
  },
  methods: {
    
  }  
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
