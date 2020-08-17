<template>
  <div
    class="card"
    :style="{
      top: coordinates.y + 'px',
      left: coordinates.x + 'px',
      transform: 'rotateY(180deg)',
    }"
    :class="{
      'display-card': cardType.display,
    }"
    draggable="true"
    @dragstart="startDrag($event)"
  >
    <img class="frontside" :src="src" alt="" />
    <img class="backside" src="../assets/backside.png" alt="" />
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from "vue-property-decorator";

interface Coordinates {
  x: number;
  y: number;
}

interface CardType {
  number: number;
  type: string;
  display: boolean;
}

@Component
export default class Card extends Vue {
  @Prop() public src: string | undefined;
  @Prop() public cardType: CardType | undefined;
  public coordinates: Coordinates = { x: 0, y: 1 };
  // private cardType: CardType;

  @Emit()
  sendStartDrag() {
    return this.cardType;
  }

  public startDrag(event: DragEvent): void {
    if (this.cardType === undefined || event.dataTransfer === null) {
      return;
    }
    event.dataTransfer.dropEffect = "move";
    event.dataTransfer.effectAllowed = "move";
    event.dataTransfer.setData("text/plain", JSON.stringify(this.cardType));
    this.sendStartDrag();
  }
}
</script>

<style lang="scss" scoped>
.card {
  position: relative;
  transform-style: preserve-3d;
}

.backside {
  position: absolute;
  top: 0;
  left: 0;
  height: auto;
  width: 120px;
  transform: rotateY(180deg);
  backface-visibility: hidden;
  opacity: 0.99;
}

.frontside {
  height: auto;
  width: 120px;
  backface-visibility: hidden;
}

.display-card {
  transform: rotateY(0deg) !important;
}
</style>
