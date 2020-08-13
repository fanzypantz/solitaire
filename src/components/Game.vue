<template>
  <div class="container">
    <Card v-for="card in cards" :key="card.pathShort" :card="card.pathLong" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Card from "@/components/Card.vue";

@Component({
  components: {
    Card,
  },
})
export default class Game extends Vue {
  // @Prop() private msg!: string;
  private cards: Array<any> = [];

  mounted() {
    this.importAll(require.context("../assets/cards/", true, /\.png$/));
  }

  private importAll(r: any): void {
    r.keys().forEach((key: any) =>
      this.cards.push({ pathLong: r(key), pathShort: key })
    );
  }
}
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  height: 100vh;
  width: 100vw;
}
</style>
