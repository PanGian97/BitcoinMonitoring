<template>
  <div id="app" class="gradient-background">
    <BitcoinPrice />
  </div>
</template>

<script>
import BitcoinPrice from "@/components/BitcoinPrice.vue";

export default {
  name: "App",
  components: {
    BitcoinPrice
  }
};
</script>

<style>
.gradient-background {
    background: linear-gradient(dodgerblue, white); /* Change the direction or colors as needed */
    height: 100vh; /* to ensure full vertical height coverage */
}

</style>
