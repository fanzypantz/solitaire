<template>
  <div
    class="container"
    @mouseup="onDrop()"
    @dragover.prevent
    @dragenter.prevent
    @mousemove="onMouseMove($event)"
  >
    <div class="top-container">
      <div class="goals-container">
        <div
          class="goals"
          v-for="(goal, index) in board.goals"
          :key="index"
          @mouseover="onMouseOver($event, 'goal', index)"
        >
          <div class="goal-card" v-if="goal.length > 0">
            <Card
              v-for="(card, index) in goal"
              :key="card.pathShort"
              :card="card"
              :cardSlot="'goal'"
              :index="index"
              @send-start-drag="receiveStartDrag"
            />
          </div>
          <div v-else>
            <img
              class="backside"
              src="../assets/backside.png"
              alt=""
              draggable="false"
            />
          </div>
        </div>
      </div>

      <div class="preview-container">
        <div
          class="preview-card"
          v-if="board.preview !== null"
          @mouseover="onMouseOver($event, 'preview')"
        >
          <Card
            :card="board.preview"
            :cardSlot="'preview'"
            :index="0"
            @send-start-drag="receiveStartDrag"
          />
        </div>
        <div v-else>
          <img
            class="backside"
            src="../assets/backside.png"
            alt=""
            draggable="false"
          />
        </div>
      </div>

      <div class="deck-container" @click="nextCard()">
        <div
          class="goal-card"
          v-if="board.deck.length > 0"
          @mouseover="onMouseOver($event, 'deck')"
        >
          <Card
            v-for="(card, index) in board.deck"
            :key="index"
            :card="card"
            :cardSlot="'deck'"
            :index="index"
            @send-start-drag="receiveStartDrag"
          />
        </div>
        <div v-else>
          <img
            class="backside"
            src="../assets/backside.png"
            alt=""
            draggable="false"
          />
        </div>
      </div>
    </div>

    <div class="columns">
      <div
        class="column"
        v-for="(column, columnIndex) in board.columns"
        :key="columnIndex"
        @mouseover="onMouseOver($event, 'column', columnIndex)"
      >
        <Card
          v-for="(card, cardIndex) in column"
          :key="cardIndex"
          :card="card"
          :cardSlot="'column'"
          :index="cardIndex"
          :columnIndex="columnIndex"
          @send-start-drag="receiveStartDrag"
        />
      </div>
    </div>

    <DragStack
      v-if="dragging.length > 0"
      :cards="dragging"
      :coordinates="cursorCoordinates"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Card from "@/components/Card.vue";
import DragStack from "@/components/DragStack.vue";

interface CardInterface {
  pathLong: string;
  pathShort: string;
  cardType: {
    number: number;
    type: string;
    display: boolean;
  };
}

interface CardData {
  cardSlot: string;
  card: CardInterface;
  columnIndex: number;
  index: number;
}

interface TargetData {
  type: string;
  index: number;
}

@Component({
  components: {
    Card,
    DragStack,
  },
})
export default class Game extends Vue {
  // @Prop() private msg!: string;
  private board: Record<string, any> = {
    deck: [],
    goals: [[], [], [], []],
    preview: null,
    columns: [[], [], [], [], [], [], []],
  };
  private dragData: CardData | undefined;
  private dragging: Array<CardInterface> = [];
  private cursorCoordinates = { x: 0, y: 0 };
  private currentTarget: TargetData = {
    type: "",
    index: -1,
  };

  mounted() {
    this.importAll(require.context("../assets/cards/", true, /\.png$/));
    this.board.deck = Game.shuffleArray(this.board.deck);
    this.initiateBoard();
  }

  get positionState() {
    if (this.dragging.length === 0) {
      return "absolute";
    } else {
      return "relative";
    }
  }

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  private importAll(r: any): void {
    // import all cards from the assets and find the correct data
    r.keys().forEach((key: any) => {
      const cardString = key.substring(2, key.length - 4).split("_");
      // Assign the data to the deck on the board
      this.board.deck.push({
        pathLong: r(key),
        pathShort: key,
        cardType: {
          number: Game.parseNumber(cardString[0]),
          type: cardString[2],
          display: false,
        },
      });
    });
  }

  private removeFirst(): void {
    this.board.deck.shift();
  }

  private removeAtIndex(
    type: string,
    column: number,
    index: number
  ): Array<CardInterface> {
    switch (type) {
      case "column":
        return this.board.columns[column].splice(index);
      case "preview":
        this.board.preview = null;
        return [];
      default:
        return [];
    }
  }

  private initiateBoard(): void {
    for (let i = 0; i < 7; i++) {
      // Reverse the columns number so that it fills up from right to left
      for (let j = 7; j > i; j--) {
        this.board.columns[j - 1].push(this.board.deck[0]);
        // Remove the first card of the deck every time, this would be the top of the deck, index 0
        this.removeFirst();
        if (j === i + 1) {
          this.board.columns[j - 1][
            this.board.columns[j - 1].length - 1
          ].cardType.display = true;
        }
      }
    }
    this.previewNext();
  }

  private static shuffleArray(
    array: Array<CardInterface>
  ): Array<CardInterface> {
    // Random shuffle to the array, no idea what magic happens here
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  // TODO: make this a computed property instead
  private static parseNumber(cardString: string): number {
    // Parse the number so we can deal with numbers only in the code
    switch (cardString) {
      case "ace":
        return 1;
      case "jack":
        return 11;
      case "queen":
        return 12;
      case "king":
        return 13;
      default:
        return parseInt(cardString);
    }
  }

  public onMouseOver(event: DragEvent, type: string, index?: number): void {
    let i;
    if (index !== undefined) {
      i = index;
    } else {
      i = -1;
    }
    this.currentTarget = {
      type: type,
      index: i,
    };
  }

  public onMouseMove(event: MouseEvent): void {
    this.cursorCoordinates = {
      x: event.clientX,
      y: event.clientY,
    };
  }

  public receiveStartDrag(dragData: CardData): void {
    this.dragData = dragData;
    switch (this.dragData.cardSlot) {
      case "column":
        this.dragging = [
          ...this.dragging,
          ...this.removeAtIndex(
            this.dragData.cardSlot,
            this.dragData.columnIndex,
            this.dragData.index
          ),
        ];
        break;
      case "preview":
        this.dragging.push(this.dragData.card);
        this.board.preview = null;
        break;
      default:
        this.dragging.push(this.dragData.card);
        this.removeAtIndex(
          this.dragData.cardSlot,
          this.dragData.columnIndex,
          this.dragData.index
        );
        break;
    }
  }

  private checkColumn(card: CardInterface): boolean {
    const currentColumn = this.board.columns[this.currentTarget.index];
    // Check if a king can be put in an empty column
    if (currentColumn.length === 0) {
      if (card.cardType.number === 13) {
        return true;
      }
    } else {
      const lastCard = currentColumn[currentColumn.length - 1];
      return (
        lastCard.cardType.number - 1 === card.cardType.number &&
        Game.checkType(card.cardType.type, lastCard.cardType.type)
      );
    }
    return false;
  }

  private checkGoal(card: CardInterface): boolean {
    const currentGoal = this.board.goals[this.currentTarget.index];
    // If the current goal is empty any card type can be added
    if (currentGoal.length > 0) {
      const lastCard = currentGoal[currentGoal.length - 1];
      // If the card is of the same type and one card higher then it can be added
      return (
        lastCard.cardType.number + 1 === card.cardType.number &&
        card.cardType.type === lastCard.cardType.type
      );
    } else {
      return true;
    }
  }

  private canDrop(): boolean {
    if (this.dragData === undefined) {
      return false;
    }
    switch (this.currentTarget.type) {
      case "column":
        return this.checkColumn(this.dragData.card);
      case "goal":
        return this.checkGoal(this.dragData.card);
      default:
        return false;
    }
  }

  public onDrop(): void {
    if (this.dragData === undefined) {
      return;
    }
    // Check if you can drop the card or not, if you can drop it, you move the currently dragged cards into
    // the new array location.
    // else put them back from where they were found
    if (this.canDrop()) {
      switch (this.currentTarget.type) {
        case "column":
          this.addCardsColumn(this.currentTarget.index);
          break;
        case "goal":
          this.addCardsGoal(this.currentTarget.index);
          break;
        default:
          break;
      }
      // if the source was the preview, draw a new card
      if (this.dragData.cardSlot === "preview") {
        this.previewNext();
      }
    } else {
      switch (this.dragData.cardSlot) {
        case "column":
          this.addCardsColumn(this.dragData.columnIndex);
          break;
        case "preview":
          this.board.preview = this.dragData.card;
          break;
        default:
          break;
      }
    }
    // Every time you move anything out of the column check if there is a card to be revealed under it
    if (this.dragData.cardSlot === "column") {
      this.checkReveal(this.dragData.columnIndex);
    }
    // Always clear the drag data for the next drag event
    this.clearDragData();
  }

  private static checkType(card1: string, card2: string): boolean {
    if (card1 === card2) {
      return false;
    }
    return !(
      (card1 === "diamonds" && card2 === "hearts") ||
      (card1 === "hearts" && card2 === "diamonds") ||
      (card1 === "spades" && card2 === "clubs") ||
      (card1 === "clubs" && card2 === "spades")
    );
  }

  private addCardsColumn(index: number): void {
    if (this.dragData !== undefined) {
      // Add the dragged cards back
      this.board.columns[index] = [
        ...this.board.columns[index],
        ...this.dragging,
      ];
    }
  }

  private addCardsGoal(index: number): void {
    if (this.dragData !== undefined && this.dragging.length === 1) {
      this.board.goals[index].push(this.dragging[0]);
    }
  }

  private clearDragData(): void {
    // Clear the drag data
    this.dragging = [];
    this.dragData = undefined;
  }

  private checkReveal(index: number): void {
    const currentColumn = this.board.columns[index];
    // if the length is 0 there is no card to reveal
    if (currentColumn.length === 0) {
      return;
    }
    if (currentColumn[currentColumn.length - 1].cardType.display === false) {
      this.revealCard(index, currentColumn.length - 1);
    }
  }

  private revealCard(columnIndex: number, cardIndex: number): void {
    this.board.columns[columnIndex][cardIndex].cardType.display = true;
  }

  private previewNext(): void {
    // Add the first preview card to the deck object
    if (this.board.deck.length > 0) {
      this.board.preview = this.board.deck[0];
      this.board.preview.cardType.display = true;
      this.removeFirst();
    }
  }

  public nextCard(): void {
    if (this.board.deck.length > 0) {
      this.board.preview.cardType.display = false;
      this.board.deck.push(this.board.preview);
      this.previewNext();
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  height: 100vh;
  width: 100vw;
}

.columns {
  width: 100%;
  height: 100%;
  display: flex;
}

.column {
  margin: 0 20px;
  position: relative;
  width: 120px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.backside {
  height: auto;
  width: 120px;
  backface-visibility: hidden;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
}

.top-container {
  height: 200px;
  display: flex;
}

.goals-container {
  display: flex;
}

.goals {
  margin: 0 20px;
  position: relative;
}

.preview-container {
  margin: 0 20px 0 180px;
}

.deck-container {
  position: relative;
  margin: 0 20px;
}
</style>
