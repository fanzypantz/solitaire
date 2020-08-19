<template>
  <div
    v-if="card !== undefined"
    class="card"
    :style="{
      top: coordinateState.y + 'px',
      left: coordinateState.x + 'px',
      transform: transform,
      position: positionState,
    }"
    :class="{
      'display-card': card.cardType.display,
    }"
    @dragstart="startDrag($event)"
  >
    <img
      class="frontside"
      :src="card.pathLong"
      alt=""
      :style="{
        boxShadow: shadowState,
      }"
    />
    <img
      class="backside"
      src="../assets/backside.png"
      alt=""
      :style="{
        boxShadow: shadowState,
      }"
    />
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from "vue-property-decorator";

interface Coordinates {
  x: number;
  y: number;
}

interface CardInterface {
  pathLong: string;
  pathShort: string;
  cardType: {
    number: number;
    type: string;
    display: boolean;
  };
}

interface CardType {
  number: number;
  type: string;
  display: boolean;
}

@Component
export default class Card extends Vue {
  @Prop() public card: CardInterface | undefined;
  @Prop() public cardSlot: string | undefined;
  @Prop() public index: number | undefined;
  @Prop() public columnIndex: number | undefined;
  public attached = false;
  // private cardType: CardType;

  @Emit()
  sendStartDrag() {
    return {
      cardSlot: this.cardSlot,
      card: this.card,
      index: this.index,
      columnIndex: this.columnIndex,
    };
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

  get shadowState() {
    if (this.cardSlot === "deck" && this.index !== 0) {
      return "";
    } else {
      return "0 0 6px rgba(0, 0, 0, 0.4)";
    }
  }

  get draggableState() {
    if (
      this.cardSlot === "goal" ||
      (this.card !== undefined && !this.card.cardType.display)
    ) {
      return false;
    } else {
      return true;
    }
  }

  public startDrag(event: DragEvent): void {
    if (
      this.card === undefined ||
      event.dataTransfer === null ||
      this.cardSlot === "goal" ||
      !this.card.cardType.display
    ) {
      event.preventDefault();
      return;
    }
    console.log("event: ", event);
    event.dataTransfer.dropEffect = "move";
    event.dataTransfer.effectAllowed = "move";
    event.dataTransfer.setData(
      "text/plain",
      JSON.stringify({
        cardSlot: this.cardSlot,
        card: this.card,
        index: this.index,
        columnIndex: this.columnIndex,
      })
    );
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
