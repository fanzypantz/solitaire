<template>
  <div class="card-container">
    <img class="card" :src="card" alt="" />
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class Card extends Vue {
  @Prop() public card: string | undefined;
}
</script>

<style lang="scss" scoped>
.card-container {
}

.card {
  position: absolute;
  top: 0;
  left: 0;
  width: auto;
  height: 200px;
}
</style>
