<template>
  <div class="hello">
    <div v-if="playing">
      <h2>Current Player: {{ currentPlayer.name }}</h2>

      <h3>Current Card: {{ currentCard }}</h3>

      <div v-if="currentCardHasInput">
        <input type="text" v-model.number="score" />
      </div>

      <div v-if="currentCardHasYesNo">
        <button @click="nextTurn(true);">Yes</button>
        <button @click="nextTurn(false);">No</button>
      </div>

      <div v-else><button @click="nextTurn();">Next Turn</button></div>

      <ol>
        <li v-for="player in sortedPlayers">
          {{ player.name }} - {{ player.score }}
        </li>
      </ol>
    </div>

    <div v-if="finished">
      <h1>Winner: {{ winner.name }}</h1>

      <button @click="restart">Restart</button>
    </div>

    <div v-if="!currentPlayer && !finished">
      <button @click="startGame">Start</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      score: 0,
      initialCards: {
        Kleeblatt: 1,
        Feuerwerk: 5,
        Stop: 10,
        Kniffel: 5,
        "Plus/Minus": 5,
        x2: 5,
        200: 5,
        300: 5,
        400: 5,
        500: 5,
        600: 5
      },
      cards: [],
      players: [
        {
          name: "Timo",
          score: 0
        },
        {
          name: "Simon",
          score: 0
        },
        {
          name: "Martin",
          score: 0
        }
      ],
      currentPlayerIndex: null,
      currentCard: null,
      winnerIndex: null,
      finished: false
    };
  },

  computed: {
    currentCardHasInput() {
      return (
        Number.isInteger(parseInt(this.currentCard)) ||
        this.currentCard === "x2" ||
        this.currentCard === "Feuerwerk"
      );
    },

    currentCardHasYesNo() {
      return ["Plus/Minus", "Kniffel", "Kleeblatt"].includes(this.currentCard);
    },

    playing() {
      return this.currentPlayer && !this.finished;
    },

    winner() {
      const winners = [...this.players]
        .filter(p => p.score >= 6000)
        .sort((a, b) => b.score - a.score);

      return winners.length ? winners[0] : null;
    },

    totalPlayers() {
      return this.players.length;
    },

    currentPlayer() {
      return this.players[this.currentPlayerIndex];
    },

    leader() {
      return this.sortedPlayers.length ? this.sortedPlayers[0] : null;
    },

    sortedPlayers() {
      return [...this.players].sort((a, b) => b.score - a.score);
    }
  },

  watch: {
    winner() {
      if (this.winner && this.winnerIndex === null) {
        this.winnerIndex = this.players.findIndex(
          p => p.name === this.winner.name
        );
      }
    },

    cards() {
      if (this.cards.length === 0) {
        this.shuffleCards();
      }
    }
  },

  methods: {
    startGame() {
      this.currentPlayerIndex = 0;
      this.players = this.shuffleArray(this.players);
      this.shuffleCards();
      this.pickCard();
    },

    pickCard() {
      const cards = [...this.cards];

      this.currentCard = cards.shift();

      this.cards = cards;
    },

    restart() {
      this.finished = false;

      this.players = this.players.map(p => {
        p.score = 0;

        return p;
      });

      this.startGame();
    },

    nextTurn(check = false) {
      let score = this.score;

      if (this.currentCard === "Plus/Minus" && check) {
        score = 1000;

        if (!this.leader || this.leader.name !== this.currentPlayer.name) {
          const leaderIndex = this.players.findIndex(
            p => p.name === this.leader.name
          );

          this.players[leaderIndex].score -= 1000;
        }
      }

      if (this.currentCard === "Kniffel" && check) {
        score = 2000;
      }

      if (this.currentCard === "Kleeblatt" && check) {
        this.players[this.currentPlayerIndex].score = 999999;
        this.finished = true;
        return;
      }

      this.players[this.currentPlayerIndex].score += score;

      this.nextPlayer();
      this.pickCard();

      this.score = 0;
    },

    nextPlayer() {
      this.currentPlayerIndex =
        this.currentPlayerIndex === this.totalPlayers - 1
          ? 0
          : this.currentPlayerIndex + 1;

      if (this.winnerIndex === this.currentPlayerIndex) {
        this.finished = true;

        return;
      }
    },

    shuffleCards() {
      const cards = Object.keys(this.initialCards).reduce((cards, c) => {
        const count = this.initialCards[c];

        for (let i = 1; i <= count; i++) {
          cards.push(c);
        }

        return cards;
      }, []);

      this.cards = this.shuffleArray(cards);
    },

    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }

      return array;
    }
  }
};
</script>

<style>
.is-leader {
  background-color: green;
}
</style>
