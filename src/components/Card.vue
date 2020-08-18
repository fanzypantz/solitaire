<template>
  <div
    class="card"
    :style="{
      top: coordinateState.y + 'px',
      left: coordinateState.x + 'px',
      transform: transform,
      position: positionState,
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
  @Prop() public cardSlot: string | undefined;
  @Prop() public index: number | undefined;
  @Prop() public attached = false;
  public coordinates: Coordinates = { x: 0, y: 0 };
  // private cardType: CardType;

  @Emit()
  sendStartDrag() {
    return this.cardType;
  }

  // mounted() {
  //   this.coordinates = { x: 0, y: (this.index || 0) * 50 };
  // }

  get coordinateState() {
    if (this.cardSlot === "goal") {
      return { x: 0, y: 0 };
    } else if (this.cardSlot === "column") {
      return { x: 0, y: (this.index || 0) * 50 };
    } else {
      return { x: 0, y: 0 };
    }
  }

  get transform() {
    if (this.index !== undefined && this.index > 0) {
      return "rotateY(180deg)";
    } else {
      return "rotateY(180deg)";
    }
  }

  get positionState() {
    if (this.index !== undefined && this.index > 0) {
      return "absolute";
    } else {
      return "relative";
    }
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
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
}

.frontside {
  height: auto;
  width: 120px;
  backface-visibility: hidden;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
}

.display-card {
  transform: rotateY(0deg) !important;
}
</style>
