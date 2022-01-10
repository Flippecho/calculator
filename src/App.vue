<template>
  <div id="app">
    <TheCalculator v-bind:postOrderExpression='postOrderExpression' v-on:expressionTreeChanged="changeExpressionTree"></TheCalculator>
    <TheTreeDisplay v-bind:expressionTree='expressionTree' v-on:postOrderExpressionChanged="changePostOrderExpression"></TheTreeDisplay>
    <TheFooter></TheFooter>
  </div>
</template>

<script>
import TheFooter from "@/components/TheFooter";
import TheCalculator from "@/components/TheCalculator";
import TheTreeDisplay from "@/components/TheTreeDisplay";

export default {
  name: 'App',
  components: {TheTreeDisplay, TheCalculator, TheFooter},
  data() {
    return {
      expressionTree: {},
      postOrderExpression: []
    }
  },
  methods: {
    changeExpressionTree(e) {
      this.expressionTree = e;
    },
    changePostOrderExpression(e) {
      this.postOrderExpression = e;
    }
  }
}

</script>

<style>

#app {
  height: 100vh;
  overflow: hidden;
}

</style>
