<template>
  <section class="slider" :style="sliderStyles">
    <header class="slider-header">
      <slot name="header" />
      <div class="slider-controls">
        <button
          class="slider-button button-previous"
          type="button"
          @click="handleScrollTo('left')"
          :disabled="isStart"
        >
          <span class="sr-only">Previous</span>
        </button>
        <button
          class="slider-button button-next"
          type="button"
          @click="handleScrollTo('right')"
          :disabled="isEnd"
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
      gap: undefined,
    },
    scrollOptions: {
      behavior: "smooth",
    },
    indexInView: [],
    isStart: true,
    isEnd: false,
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
    gap: {
      type: String,
      default: undefined,
    },
    breakpoints: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    uniqueSortedIndexes({ indexInView }) {
      return [...new Set(indexInView)].sort((a, b) => a - b);
    },
    sliderStyles({
      sliderOptions: { width = this.width, peak = this.peak, gap = this.gap },
    }) {
      return {
        "--width": width,
        "--peak": peak,
        "--gap": gap,
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

        if (!matches) {
          return;
        }

        for (const key in breakpoint) {
          Object.assign(this.sliderOptions, {
            [key]: breakpoint[key] || this.sliderOptions[key],
          });
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
      if (!this.intersectionObserver) {
        return;
      }

      this.intersectionObserver.disconnect();
    },
    handleIntersectionObserve(entries) {
      const contentSlot = this.$slots.content;

      for (const entry of entries) {
        const entryIndex = contentSlot.findIndex(
          (element) => element.elm === entry.target
        );

        if (entry.isIntersecting && entry.intersectionRatio > 0.5) {
          this.indexInView.push(entryIndex);
        } else {
          this.indexInView = this.indexInView.filter(
            (index) => index !== entryIndex
          );
        }
      }

      this.updateControls();
    },
    handleScrollTo(direction) {
      let targetItem;

      const contentSlot = this.$slots.content;
      const newSlideIndex = this.uniqueSortedIndexes[0];

      if (direction === "right") {
        targetItem =
          contentSlot[newSlideIndex + this.uniqueSortedIndexes.length];

        if (!targetItem) {
          this.scrollToEnd();
          return;
        }
      }

      if (direction === "left") {
        targetItem =
          contentSlot[newSlideIndex - this.uniqueSortedIndexes.length];

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
    updateControls() {
      const contentSlot = this.$slots.content;

      this.isStart = this.indexInView.includes(0) === true;
      this.isEnd = this.indexInView.includes(contentSlot.length - 1) === true;
    },
  },
};
</script>

<style scoped>
.slider {
  --width: 100vw;
  --peak: 0vw;
  --gap: 0;
}

.slider-header {
  margin: clamp(1.5rem, 3vw, 2rem);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.slider-header h1 {
  font-size: clamp(2rem, 4vw, 3rem);
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
  border-top-width: 0.125em;
  border-right-width: 0.125em;
  border-style: solid;
  border-color: var(--color-black);
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

.button-next {
  right: 0.125em;
  transform: translate(0, -50%) rotate(45deg);
}

.button-previous {
  left: 0.125em;
  transform: translate(0, -50%) rotate(-135deg);
}

.slider-content {
  display: flex;
  gap: var(--gap);
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
  border-top-width: 1px;
  border-bottom-width: 1px;
  border-style: solid;
  border-color: var(--color-black);
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
