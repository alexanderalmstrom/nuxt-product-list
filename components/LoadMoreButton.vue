<template>
  <button type="button" @click="$emit('click')">Load more</button>
</template>

<script>
export default {
  name: "LoadMoreButton",
};
</script>

<style scoped>
.load-more-button {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  padding-inline: 2em;
  height: 3.5em;
  background-color: #ffffff;
  box-shadow: 0 0 0 1px #000000;
  transition-property: background-color, color;
  transition-duration: 150ms;
  transition-timing-function: ease;
}

.load-more-button:hover {
  background-color: hsl(0, 0%, 95%);
}
</style>
