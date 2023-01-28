<template>
  <section class="slider" :style="sliderStyles">
    <header class="slider-header">
      <slot name="header" />
      <div class="slider-controls">
        <button
          class="slider-button previous-button"
          type="button"
          @click="handleScrollTo('left')"
        >
          <span class="sr-only">Previous</span>
        </button>
        <button
          class="slider-button next-button"
          type="button"
          @click="handleScrollTo('right')"
        >
          <span class="sr-only">Next</span>
        </button>
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
      width: undefined,
      peak: undefined,
    },
    visibleIndexes: [],
    scrollOptions: {
      behavior: "smooth",
    },
  }),
  props: {
    width: {
      type: String,
      default: undefined,
    },
    peak: {
      type: String,
      default: undefined,
    },
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
        "--width": sliderOptions.width || this.width,
        "--peak": sliderOptions.peak || this.peak,
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
      if (!this.breakpoints.length) {
        return;
      }

      this.resizeObserver = new ResizeObserver(this.handleResizeObserve);
      this.resizeObserver.observe(this.$refs.content);
    },
    destroyResizeObserver() {
      if (!this.resizeObserver) {
        return;
      }

      this.resizeObserver.disconnect();
    },
    handleResizeObserve() {
      for (const breakpoint of this.breakpoints) {
        const { matches } = window.matchMedia(breakpoint.media);

        if (matches) {
          if (breakpoint.width) {
            this.sliderOptions.width = breakpoint.width;
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
      let targetItem;
      const contentSlot = this.$slots.content;
      const newSlideIndex = this.uniqueSortedItems[0];

      if (direction === "right") {
        targetItem = contentSlot[newSlideIndex + this.uniqueSortedItems.length];

        if (!targetItem) {
          this.scrollToEnd();
          return;
        }
      }

      if (direction === "left") {
        targetItem = contentSlot[newSlideIndex - this.uniqueSortedItems.length];

        if (!targetItem) {
          this.scrollToStart();
          return;
        }
      }

      this.scrollToElement({ element: targetItem.elm });
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
      const lastItem = contentSlot[contentSlot.length - 1];
      this.scrollToElement({ element: lastItem.elm });
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
  --width: 100vw;
  --peak: 0vw;
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
  width: 3rem;
  position: relative;
}

.slider-button {
  font-size: 0.8rem;
  width: 1em;
  height: 1em;
  border-top: 0.125em solid #000000;
  border-right: 0.125em solid #000000;
  position: absolute;
}

.slider-button:before {
  content: "";
  width: 100%;
  height: 100%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(45deg) scale(2);
}

.next-button {
  right: 0.125em;
  transform: translate(0, -50%) rotate(45deg);
}

.previous-button {
  left: 0.125em;
  transform: translate(0, -50%) rotate(-135deg);
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
  min-width: calc(var(--width) - var(--peak));
  flex-grow: 1;
  flex-shrink: 0;
}
</style>
