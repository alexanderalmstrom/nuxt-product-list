<template>
  <section class="slider">
    <header class="slider-header">
      <slot name="header" />
    </header>
    <div class="slider-content">
      <slot name="content" />
    </div>
  </section>
</template>

<script>
export default {
  name: "SliderComponent",
  props: {
    items: {
      type: Array,
      default: () => [],
    },
  },
};
</script>

<style scoped>
.slider-header {
  margin: 1.5rem;
}

.slider-content {
  --slide-width: calc(75vw - 1.5rem);

  display: flex;
  gap: 1px;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
  border-top: 1px solid;
  border-bottom: 1px solid;
  border-color: #000000;
}

.slider-content > * {
  scroll-snap-align: start;
}

.slider-content::-webkit-scrollbar {
  display: none;
}

.slider-content ::v-deep .product-card {
  min-width: var(--slide-width);
  flex-grow: 1;
  flex-shrink: 0;
}

@media (min-width: 1024px) {
  .slider-content ::v-deep .product-card {
    --slide-width: calc(25vw - 1.5rem);
  }
}
</style>
