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
    <span class="game-state" v-if="gameOverMessage.length">{{
      gameOverMessage
    }}</span>
    <div class="state-container game-state" v-else>
      <span>You are {{ myCharacter }}</span>
      <span>Turn: {{ turn }}</span>
    </div>
    <span class="room-number" @click="copyRoomURL">Room: {{ room }}</span>

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
import Row from "@/components/Row.vue";
import io from "socket.io-client";

const socket = io("https://murmuring-fjord-44762.herokuapp.com/");

function getBoard() {
  return Array.from({ length: 3 }, () => Array.from({ length: 3 }, () => ""));
}

interface IWinState {
  isTie: boolean;
  winner: string;
}

export default {
  data() {
    return {
      myCharacter: "",
      board: getBoard(),
      room: "",
      turn: "",
      gameOverMessage: "",
      gameOverColor: "",
      notifications: [],
    };
  },
  components: { Row },
  beforeMount() {
    const params = new URLSearchParams(window.location.search);
    const currentRoom = params.get("room");

    const getGameOverBackgroundColor = (winState: IWinState) => {
      if (winState.isTie) return "yellow";
      return winState.winner === this.myCharacter ? "green" : "red";
    };

    socket.on("connect", () => {
      if (currentRoom) socket.emit("join-room", currentRoom);
      else socket.emit("new-room");

      socket.on("join-error", ({ message }: { message: string }) =>
        this.notify(message)
      );

      socket.on("game-over", (winState: IWinState) => {
        if (winState.isTie) this.gameOverMessage = "TIE - No one wins!";
        else this.gameOverMessage = `${winState.winner} WINS!`;
        this.gameOverColor = getGameOverBackgroundColor(winState);
      });

      socket.on(
        "joined-room",
        (
          data: {
            room: string;
            char: string;
            turn: string;
            board: string[][];
            gameOver: boolean;
            winner: string;
          } & IWinState
        ) => {
          console.log("joined-room", data.room);
          params.set("room", data.room);
          this.myCharacter = data.char;
          this.room = data.room;
          this.board = data.board;
          this.turn = data.turn;
          if (data.gameOver) {
            if (data.isTie) this.gameOverMessage = "TIE - No one wins!";
            else this.gameOverMessage = `${data.winner} WINS!`;
            this.gameOverColor = getGameOverBackgroundColor(data);
          }
          // if (!currentRoom) window.location.search = params.toString();
        }
      );

      socket.on("play-error", ({ message }: { message: string }) =>
        this.notify(message)
      );

      socket.on(
        "tile-clicked",
        (data: { location: { x: number; y: number }; char: string }) => {
          console.log("tile-clicked", data);
          const { location, char } = data;
          this.turn = this.turn === "X" ? "O" : "X";
          this.board[location.x].splice(location.y, 1, char);
        }
      );
      socket.once("player-joined", () => this.notify("Player joined"));
    });
    socket.on("disconnect", () => console.log("disconnected!"));
  },
  methods: {
    copyRoomURL() {
      this.notifications.push("Room URL copied");
      const el = document.createElement("input");
      const url = new URL(window.location);
      url.searchParams.set("room", this.room);
      el.value = url.href;
      el.style.left = "-9999px";
      document.body.appendChild(el);
      el.select();
      document.execCommand("copy");
      document.body.removeChild(el);
    },
    notify(message) {
      this.notifications.push(message);
    },
    onTileClicked(e) {
      socket.emit("tile-clicked", {
        room: this.room,
        location: e,
        char: this.myCharacter,
      });
    },
  },
};
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
