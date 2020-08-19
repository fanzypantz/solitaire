<template>
  <div
    class="drag-stack-container"
    :style="{
      top: coordinates.y + 'px',
      left: coordinates.x + 'px',
    }"
  >
    <Card
      v-for="(card, index) in cards"
      :key="index"
      :card="card"
      :cardSlot="'dragStack'"
      :index="index"
    />
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import Card from "@/components/Card.vue";

interface CardType {
  number: number;
  type: string;
  display: boolean;
}

@Component({
  components: {
    Card,
  },
})
export default class DragStack extends Vue {
  @Prop() public cards: Array<CardType> | undefined;
  @Prop() public coordinates: object | undefined;

  mounted() {
    console.log("cards: ", this.cards);
  }
}
</script>

<style lang="scss" scoped>
.drag-stack-container {
  position: fixed;
  z-index: 100;
  transform: translate(-50%, -50%);
}
</style>
