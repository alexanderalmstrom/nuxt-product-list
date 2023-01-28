<template>
  <section class="slider" :style="sliderStyles">
    <header class="slider-header">
      <slot name="header" />
      <div class="slider-controls">
        <button type="button" @click="handleScrollTo('left')">Previous</button>
        <button type="button" @click="handleScrollTo('right')">Next</button>
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
    intersectionObserver: undefined,
    resizeObserver: undefined,
    sliderOptions: {
      items: undefined,
      peak: undefined,
    },
    visibleIndexes: [],
    scrollOptions: {
      behavior: "smooth",
    },
  }),
  props: {
    breakpoints: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    uniqueSortedItems({ visibleIndexes }) {
      return [...new Set(visibleIndexes)].sort((a, b) => a - b);
    },
    sliderStyles({ sliderOptions }) {
      return {
        "--items": sliderOptions.items,
        "--peak": sliderOptions.peak,
      };
    },
  },
  mounted() {
    this.initializeIntersectionObserver();
    this.initializeResizeObserver();
  },
  beforeDestroy() {
    this.destroyIntersectionObserver();
    this.destroyResizeObserver();
  },
  methods: {
    initializeResizeObserver() {
      this.resizeObserver = new ResizeObserver(this.handleResizeObserve);
      this.resizeObserver.observe(this.$refs.content);
    },
    destroyResizeObserver() {
      this.resizeObserver.disconnect();
    },
    handleResizeObserve() {
      for (const breakpoint of this.breakpoints) {
        const { matches } = window.matchMedia(breakpoint.media);

        if (matches) {
          if (breakpoint.items) {
            this.sliderOptions.items = breakpoint.items;
          }

          if (breakpoint.peak) {
            this.sliderOptions.peak = breakpoint.peak;
          }
        }
      }
    },
    initializeIntersectionObserver() {
      this.intersectionObserver = new IntersectionObserver(
        this.handleIntersectionObserve,
        {
          root: this.$el,
          threshold: [0, 0.25, 0.5, 0.75, 1],
        }
      );

      this.$slots.content.forEach((item) => {
        this.intersectionObserver.observe(item.elm);
      });
    },
    destroyIntersectionObserver() {
      this.resizeObserver.disconnect();
    },
    handleIntersectionObserve(entries) {
      const contentSlot = this.$slots.content;

      for (const entry of entries) {
        const entryIndex = contentSlot.findIndex(
          (element) => element.elm === entry.target
        );

        if (entry.isIntersecting && entry.intersectionRatio > 0.5) {
          this.visibleIndexes.push(entryIndex);
        } else {
          this.visibleIndexes = this.visibleIndexes.filter(
            (index) => index !== entryIndex
          );
        }
      }
    },
    handleScrollTo(direction) {
      let newSlideItem;
      const contentSlot = this.$slots.content;
      const newSlideIndex = this.uniqueSortedItems[0];

      if (direction === "right") {
        newSlideItem =
          contentSlot[newSlideIndex + this.uniqueSortedItems.length];

        if (!newSlideItem) {
          this.scrollToEnd();
          return;
        }
      }

      if (direction === "left") {
        newSlideItem =
          contentSlot[newSlideIndex - this.uniqueSortedItems.length];

        if (!newSlideItem) {
          this.scrollToStart();
          return;
        }
      }

      this.scrollToElement({ element: newSlideItem.elm });
    },
    scrollToStart() {
      const contentRef = this.$refs.content;
      contentRef.scrollTo({
        left: 0,
        ...this.scrollOptions,
      });
    },
    scrollToEnd() {
      const contentSlot = this.$slots.content;
      const lastSlideItem = contentSlot[contentSlot.length - 1];
      this.scrollToElement({ element: lastSlideItem.elm });
    },
    scrollToElement({ element }) {
      const contentRef = this.$refs.content;
      contentRef.scrollTo({
        left: element.offsetLeft,
        ...this.scrollOptions,
      });
    },
  },
};
</script>

<style scoped>
.slider {
  --peak: 4vw;
  --slide-peak: calc(var(--peak) / var(--items));
  --slide-width: calc(100vw / var(--items) - var(--slide-peak));
}

.slider-header {
  margin: clamp(1.5rem, 3vw, 2rem);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.slider-header h1 {
  font-size: clamp(2rem, 6vw, 3rem);
  margin-bottom: 0;
}

.slider-controls {
  display: flex;
  column-gap: 1rem;
}

.slider-controls button {
  font-size: clamp(0.875rem, 1vw, 1.125rem);
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
  min-width: var(--slide-width, 100vw);
  flex-grow: 1;
  flex-shrink: 0;
}
</style>
