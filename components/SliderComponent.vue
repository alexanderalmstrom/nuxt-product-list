<template>
  <section class="slider">
    <header class="slider-header">
      <slot name="header" />
      <div class="slider-controls">
        <button type="button" @click="handleScroll('left')">Previous</button>
        <button type="button" @click="handleScroll('right')">Next</button>
      </div>
    </header>
    <div ref="content" class="slider-content">
      <slot name="content" />
    </div>
  </section>
</template>

<script>
export default {
  name: "SliderComponent",
  data: () => ({
    observer: undefined,
    visibleItems: [],
    scrollOptions: {
      behavior: "smooth",
    },
  }),
  computed: {
    uniqueSortedItems() {
      return [...new Set(this.visibleItems)].sort((a, b) => a - b);
    },
  },
  mounted() {
    this.initializeObserver();
  },
  beforeDestroy() {
    this.observer.disconnect();
  },
  methods: {
    initializeObserver() {
      this.observer = new IntersectionObserver(this.handleObserve, {
        root: this.$el,
        threshold: [0, 0.25, 0.5, 0.75, 1],
      });

      this.$slots.content.forEach((item) => {
        this.observer.observe(item.elm);
      });
    },
    handleObserve(entries) {
      const contentSlot = this.$slots.content;

      for (const entry of entries) {
        const entryIndex = contentSlot.findIndex(
          (element) => element.elm === entry.target
        );

        if (entry.isIntersecting && entry.intersectionRatio > 0.5) {
          this.visibleItems.push(entryIndex);
        } else {
          this.visibleItems = this.visibleItems.filter(
            (index) => index !== entryIndex
          );
        }
      }
    },
    handleScroll(direction) {
      let newSlideItem;
      const contentRef = this.$refs.content;
      const contentSlot = this.$slots.content;
      const newSlideIndex = this.uniqueSortedItems[0];
      const firstSlideItem = contentSlot[0];
      const lastSlideItem = contentSlot[contentSlot.length - 1];

      if (direction === "right") {
        newSlideItem =
          contentSlot[newSlideIndex + this.uniqueSortedItems.length];

        if (!newSlideItem) {
          const offsetLeft = lastSlideItem.elm.offsetLeft;
          contentRef.scrollTo({ left: offsetLeft, ...this.scrollOptions });
          return;
        }
      }

      if (direction === "left") {
        newSlideItem =
          contentSlot[newSlideIndex - this.uniqueSortedItems.length];

        if (!newSlideItem) {
          const offsetLeft = firstSlideItem.elm.offsetLeft;
          contentRef.scrollTo({ left: offsetLeft, ...this.scrollOptions });
          return;
        }
      }

      const offsetLeft = newSlideItem.elm.offsetLeft;
      contentRef.scrollTo({ left: offsetLeft, ...this.scrollOptions });
    },
  },
};
</script>

<style scoped>
.slider {
  --peak: 1.5rem;
  --slide-width: calc(75vw - var(--peak));
}

.slider-header {
  margin: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.slider-header h1 {
  margin-bottom: 0;
}

.slider-controls button {
  font-size: 0.875rem;
  padding-top: 0.2em;
  padding-bottom: 0.2em;
  border-bottom: 1px solid #000000;
}

.slider-content {
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

.slider-content ::v-deep > * {
  scroll-snap-align: start;
  min-width: var(--slide-width);
  flex-grow: 1;
  flex-shrink: 0;
}

@media (min-width: 1024px) {
  .slider-content ::v-deep > * {
    --slide-width: calc(25vw - var(--peak));
  }
}
</style>
