<template>
  <tr>
    <Tile
      v-for="(tile, index) in row"
      :key="index"
      :rowNumber="rowNumber"
      :columnNumber="index"
      :content="tile"
    />
  </tr>
</template>

<script lang="ts">
import Tile from "@/components/Tile.vue";
export default {
  props: {
    row: { type: Array, required: true },
    rowNumber: { type: Number, required: true },
  },
  components: { Tile },
};
</script>
