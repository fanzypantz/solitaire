<template>
  <div
    class="container"
    @drop="onDrop($event)"
    @dragover.prevent
    @dragenter.prevent
    @mousemove="onMouseMove($event)"
  >
    <div class="top-container">
      <div class="goals-container">
        <div class="goals" v-for="(goal, index) in board.goals" :key="index">
          <div class="goal-card" v-if="goal.length > 0">
            <Card
              v-for="(card, index) in goal"
              :key="card.pathShort"
              :src="card.pathLong"
              :cardType="card.cardType"
              :cardSlot="'goal'"
              :index="index"
              @send-start-drag="receiveStartDrag"
            />
          </div>
          <div v-else>
            <img class="backside" src="../assets/backside.png" alt="" />
          </div>
        </div>
      </div>
    </div>
    <div class="columns">
      <div class="column" v-for="(column, index) in board.columns" :key="index">
        <Card
          v-for="(card, index) in column"
          :key="card.pathShort"
          :src="card.pathLong"
          :cardType="card.cardType"
          :cardSlot="'column'"
          :index="index"
          @send-start-drag="receiveStartDrag"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Card from "@/components/Card.vue";

interface CardInterface {
  pathLong: string;
  pathShort: string;
  cardType: {
    number: number;
    type: string;
    display: boolean;
  };
}

@Component({
  components: {
    Card,
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
  private dragging: CardInterface | undefined;

  mounted() {
    console.log("game mounted");
    this.importAll(require.context("../assets/cards/", true, /\.png$/));
    this.board.deck = Game.shuffleArray(this.board.deck);
    this.initiateBoard();
    console.log("this.board: ", this.board);
  }

  get positionState() {
    if (this.dragging !== undefined) {
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
    // Add the first preview card to the deck object
    this.board.preview = this.board.deck[0];
    this.removeFirst();
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

  public receiveStartDrag(cardType: object): void {
    console.log("test  emit: ", cardType);
  }

  public onDrop(event: DragEvent): void {
    if (event.dataTransfer === null) {
      return;
    }
    const cardID = JSON.parse(event.dataTransfer.getData("text/plain"));
    console.log("cardid: ", cardID);
  }

  public onMouseMove(event: MouseEvent): void {
    // console.log("test: ");
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
}

.goals-container {
  display: flex;
}

.goals {
  margin: 0 20px;
}
</style>
