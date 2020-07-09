<template>
  <td
    class="tile"
    @click="$parent.$emit('tile-clicked', { x: rowNumber, y: columnNumber })"
    :rowNumber="rowNumber"
    :columnNumber="columnNumber"
    :class="{ 'blank-tile': content.length }"
  >
    {{ content }}
  </td>
</template>

<script>
export default {
  props: {
    content: { type: String, required: true },
    rowNumber: { type: Number, required: true },
    columnNumber: { type: Number, required: true },
  },
};
</script>

<style>
.tile {
  width: 100px;
  height: 100px;
  text-align: center;
  font-size: 30px;
  border: 1px solid black;
  border-radius: 5px;
  background-color: #333;
  color: #fff;
  user-select: none;
  cursor: pointer;
}

.tile:hover:not(.blank-tile) {
  background-color: #222;
}

.blank-tile {
  background-color: #444;
}
</style>
