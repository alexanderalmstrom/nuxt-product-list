<template>
  <section class="slider" :style="sliderStyles">
    <header class="slider-header">
      <slot name="header" />
      <div class="slider-controls">
        <button
          class="slider-button button-previous"
          type="button"
          :disabled="isStart"
          @click="handleScrollTo('left')"
        >
          <span class="sr-only">Previous</span>
        </button>
        <button
          class="slider-button button-next"
          type="button"
          :disabled="isEnd"
          @click="handleScrollTo('right')"
        >
          <span class="sr-only">Next</span>
        </button>
      </div>
    </header>
    <div ref="scroll" class="slider-scroll">
      <slot name="content" />
    </div>
  </section>
</template>

<script>
export default {
  name: "SliderComponent",
  props: {
    gap: {
      type: String,
      default: "0rem",
    },
    width: {
      type: String,
      default: "100vw",
    },
    peek: {
      type: String,
      default: "0vw",
    },
    breakpoints: {
      type: Array,
      default: () => [],
    },
  },
  data: () => ({
    intersectionObserver: undefined,
    resizeObserver: undefined,
    sliderOptions: {
      gap: undefined,
      width: undefined,
      peek: undefined,
    },
    scrollOptions: {
      behavior: "smooth",
    },
    itemsInView: new Set(),
    isStart: true,
    isEnd: false,
  }),
  computed: {
    sliderStyles({
      sliderOptions: { gap = this.gap, width = this.width, peek = this.peek },
    }) {
      return {
        "--slider-gap": gap,
        "--slide-item-width": width,
        "--slide-item-peek": peek,
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
      this.resizeObserver.observe(this.$refs.scroll);
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
          threshold: [0, 0.5, 1],
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

      for (const { target, isIntersecting, intersectionRatio } of entries) {
        const entry = contentSlot.find((element) => element.elm === target);
        const index = contentSlot.indexOf(entry);

        if (isIntersecting && intersectionRatio > 0.5) {
          this.itemsInView.add(index);
        } else {
          this.itemsInView.delete(index);
        }
      }

      this.updateControls();
    },
    handleScrollTo(direction) {
      let item;

      const contentSlot = this.$slots.content;
      const itemsInView = [...this.itemsInView].sort((a, b) => a - b);
      const newSlideIndex = [...itemsInView][0];

      if (direction === "right") {
        item = contentSlot[newSlideIndex + this.itemsInView.size];

        if (!item) {
          this.scrollToEnd();
          return;
        }
      }

      if (direction === "left") {
        item = contentSlot[newSlideIndex - this.itemsInView.size];

        if (!item) {
          this.scrollToStart();
          return;
        }
      }

      this.scrollToElement({ element: item.elm });
    },
    scrollToStart() {
      const scrollRef = this.$refs.scroll;

      scrollRef.scrollTo({
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
      const scrollRef = this.$refs.scroll;

      scrollRef.scrollTo({
        left: element.offsetLeft,
        ...this.scrollOptions,
      });
    },
    updateControls() {
      const contentSlot = this.$slots.content;

      this.isStart = this.itemsInView.has(0);
      this.isEnd = this.itemsInView.has(contentSlot.length - 1);
    },
  },
};
</script>

<style scoped>
.slider-scroll {
  display: flex;
  gap: var(--slider-gap);
  overflow-x: auto;
}

.slider-scroll ::v-deep > * {
  min-width: calc(var(--slide-item-width) - var(--slide-item-peek));
}
</style>
