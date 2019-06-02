<template>
  <div class="cosmosLedger">
    <h3>Detect device</h3>
    <h3>Retrieve balance + delegations</h3>
    <h3>Delegate</h3>
  </div>
</template>

<script>
export default {
  name: 'CosmosLedger',
  props: {
    restUrl : String,
    someProp: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
</style>
