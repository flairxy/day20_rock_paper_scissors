<template>
  <div>
    <div class="container">
      <h1 class="text-center">Rock Paper Scissors</h1>

      <p>State: {{ states[game.state] }}</p>

      <div v-if="game.state === '1'">
        <p>Bet: {{ game.bet }}</p>
        <div>
          <h2>Players</h2>
          <ul>
            <li v-for="player in game.players" :key="player">{{ player }}</li>
          </ul>
        </div>
      </div>

      <div class="row" v-if="game.state === '0'">
        <div class="col-sm-12">
          <h2>Create Game</h2>
          <form @submit.prevent="createGame">
            <div class="form-group">
              <label htmlFor="participant">Participant</label>
              <input
                type="text"
                v-model="participant"
                class="form-control"
                id="participant"
              />
            </div>
            <div class="form-group">
              <label htmlFor="bet">Bet</label>
              <input v-model="bet" type="text" class="form-control" id="bet" />
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
        </div>
      </div>

      <div
        class="row"
        v-if="game.state === '1' && game.players[1] === activeAccount"
      >
        <div class="col-sm-12">
          <h2>Bet</h2>
          <button
            @click.prevent="joinGame"
            type="submit"
            class="btn btn-primary"
          >
            Submit
          </button>
        </div>
      </div>

      <div class="row" v-if="game.state === '2'">
        <div class="col-sm-12">
          <h2>Commit move</h2>
          <form @submit.prevent="commitMove">
            <div class="form-group">
              <label htmlFor="move">Move</label>
              <select class="form-control" id="move" v-model="move">
                <option value="1">Rock</option>
                <option value="2">Paper</option>
                <option value="3">Scissors</option>
              </select>
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
        </div>
      </div>

      <div class="row" v-if="game.state === '3'">
        <div class="col-sm-12">
          <h2>Reveal move</h2>
          <button
            @click.prevent="revealMove"
            type="submit"
            class="btn btn-primary"
          >
            Submit
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
export default {
  data: () => ({
    states: ["IDLE", "CREATED", "JOINED", "COMMITED", "REVEALED"],
    game: {
      state: "0",
    },
    participant: "",
    bet: 0,
    gameId: undefined,
    move: undefined,
    movesData: [],
  }),
  computed: {
    ...mapGetters("accounts", ["activeAccount"]),
    ...mapGetters("drizzle", ["drizzleInstance"]),
  },
  methods: {
    createGame() {
      this.drizzleInstance.contracts.RockPaperScissors.methods
        .createGame(this.participant)
        .send({ from: this.activeAccount, value: this.bet })
        .then(() => {
          this.init();
        });
    },
    joinGame() {
      let game = this.game;
      this.drizzleInstance.contracts.RockPaperScissors.methods
        .joinGame(this.gameId)
        .send({ from: this.activeAccount, value: game.bet })
        .then(() => {
          this.init();
        });
    },
    commitMove() {
      let game = this.game;
      let salt = Math.floor(Math.random() * 1000);
      this.drizzleInstance.contracts.RockPaperScissors.methods
        .commitMove(game.id, this.move, salt)
        .send()
        .then(() => {
          this.movesData.push({
            account: this.activeAccount,
            salt: salt,
            move: this.move,
          });
          this.init();
        });
    },
    revealMove() {
      let game = this.game;
      let movesData = this.movesData;
      let move;
      movesData.map((data) => {
        if (this.activeAccount == data.account) {
          move = data;
        }
      });
      this.drizzleInstance.contracts.RockPaperScissors.methods
        .revealMove(game.id, move.id, move.salt)
        .send()
        .then(() => {
          this.init();
        });
    },
    async init() {
      const contract = this.drizzleInstance.contracts.RockPaperScissors;
      let gameId = await contract.methods
        .gameId()
        .call()
        .then((res) => {
          return res;
        });
      let n_gameId = parseInt(gameId);
      n_gameId = n_gameId > 0 ? n_gameId - 1 : n_gameId;
      this.gameId = n_gameId;
      let game = await contract.methods
        .getGame(n_gameId)
        .call()
        .then((res) => {
          return res;
        });
      this.game = {
        id: game[0],
        bet: game[1],
        players: game[2],
        state: game[3],
      };
    },
  },
  created() {
    this.init();
  },
};
</script>
