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
