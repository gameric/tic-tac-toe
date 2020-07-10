<template>
  <div class="board-container">
    <h1 class="title">Tic-Tac-Toe</h1>
    <table class="board-table" :style="{ 'background-color': gameOverColor }">
      <tbody>
        <Row
          v-for="(row, index) in board"
          v-on:tile-clicked="onTileClicked"
          :row="row"
          :key="index"
          :rowNumber="index"
        />
      </tbody>
    </table>
    <span class="game-state" v-if="gameOver">{{ gameOverMessage }}</span>
    <div class="state-container game-state" v-else>
      <span>You are {{ myChar }}</span>
      <span>Turn: {{ turn }}</span>
    </div>
    <span class="room-number" @click="copyRoomURL">Room: {{ room }}</span>
    <button class="game-state" @click="onNewGame" v-if="gameOver">
      New game?
    </button>

    <div class="notifications">
      <span
        v-for="(notification, index) in notifications"
        :key="index"
        class="notification"
        >{{ notification }}</span
      >
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Row from "@/components/Row.vue";
import io from "socket.io-client";
import {
  IMyGameState,
  IGameState,
  IWinState,
  getOpponent,
  PLAYER,
  ITileClickedEvent,
  newBoard,
  IPoint,
} from "tic-tac-toe-shared";

const socket = io("http://192.168.1.7:5000/");

export default Vue.extend({
  data() {
    return {
      board: newBoard(),
      turn: "",
      room: "",
      myChar: "",
      gameOver: false,
      gameOverColor: "",
      gameOverMessage: "",
      notifications: [""],
    };
  },
  components: { Row },
  beforeMount() {
    const params = new URLSearchParams(window.location.search);
    const currentRoom = params.get("room");

    const getGameOverBackgroundColor = (winState: IWinState) => {
      if (winState.isTie) return "yellow";
      return winState.winner === this.myChar ? "green" : "red";
    };

    socket.on("connect", () => {
      if (currentRoom) socket.emit("join-room", currentRoom);
      else socket.emit("new-room");

      socket.on("err", ({ message }: { message: string }) =>
        this.notify(message)
      );

      socket.on("game-over", (winState: IWinState) => {
        if (winState.isTie) this.gameOverMessage = "TIE - No one wins!";
        else this.gameOverMessage = `${winState.winner} WINS!`;
        this.gameOver = true;
        this.gameOverColor = getGameOverBackgroundColor(winState);
      });

      socket.on("new-game", (data: IGameState) => {
        this.room = data.room;
        this.board = data.board;
        this.turn = data.turn;
        this.gameOver = data.gameOver;
        this.gameOverColor = "";
        this.notify("A new game started");
      });

      socket.on("joined-room", (data: IMyGameState) => {
        this.myChar = data.myChar;
        this.room = data.room;
        this.board = data.board;
        this.turn = data.turn;
        this.gameOver = data.gameOver;
        if (data.gameOver) {
          if (data.isTie) this.gameOverMessage = "TIE - No one wins!";
          else this.gameOverMessage = `${data.winner} WINS!`;
          this.gameOverColor = getGameOverBackgroundColor(data);
        }
        // if (!currentRoom) window.location.search = params.toString();
      });

      socket.on("tile-clicked", (data: ITileClickedEvent) => {
        const { point, char } = data;
        this.turn = getOpponent(this.turn as PLAYER);
        this.board[point.x].splice(point.y, 1, char);
      });
      socket.once("player-joined", () => this.notify("Player joined"));
    });
    socket.on("disconnect", () => this.notify("disconnected"));
  },
  methods: {
    onNewGame() {
      socket.emit("new-game");
    },
    copyRoomURL() {
      this.notifications.push("Room URL copied");
      const el = document.createElement("input");
      const url = new URL(window.location.toString());
      url.searchParams.set("room", this.room);
      el.value = url.href;
      el.style.left = "-9999px";
      document.body.appendChild(el);
      el.select();
      document.execCommand("copy");
      document.body.removeChild(el);
    },
    notify(message: string) {
      this.notifications.push(message);
    },
    onTileClicked(point: IPoint) {
      socket.emit("tile-clicked", { point, char: this.myChar });
    },
  },
});
</script>

<style>
.board-container {
  background-color: #222;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.state-container {
  margin: 0 auto;
  min-width: 300px;
  justify-content: space-around;
  display: flex;
}

.board-table {
  border: 1px solid black;
  border-radius: 5px;
}

.title {
  color: #ddd;
}

.room-number {
  position: fixed;
  bottom: 1%;
  left: 50%;
  transform: translateX(-50%);
  font-size: 10px;
  color: #333;
  background-color: #888;
  padding: 3px;
  border-radius: 3px;
}

.notifications {
  position: fixed;
  display: flex;
  flex-direction: column-reverse;
  top: 1%;
  left: 50%;
  width: 20%;
  transform: translateX(-50%);
}

.notification {
  font-size: 10px;
  color: #888;
  background-color: #333;
  padding: 3px;
  border-radius: 3px;
  margin-top: 3px;
  text-align: center;
}

.hidden {
  visibility: hidden;
  opacity: 0;
  transition: visibility 2s linear, opacity 1.5s linear;
}

.game-state {
  border-radius: 5px;
  background-color: #555;
  color: #ddd;
  padding: 10px;
  font-size: 20px;
  margin-top: 20px;
}
</style>
