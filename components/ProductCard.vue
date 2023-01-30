<template>
  <article class="product-card">
    <ImageComponent
      class="product-card-image"
      :src="image"
      :lazyload="!isVisible"
      :alt="name"
    />
    <div class="product-card-content">
      <h3 class="product-card-name">{{ name }}</h3>
      <p class="product-card-price">{{ price }}</p>
    </div>
  </article>
</template>

<script>
import ImageComponent from "./ImageComponent.vue";

export default {
  name: "ProductCard",
  components: { ImageComponent },
  props: {
    image: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    price: {
      type: String,
      required: true,
    },
  },
  data: () => ({
    observer: undefined,
    isVisible: false,
  }),
  mounted() {
    this.initializeObserver();
  },
  beforeDestroy() {
    this.observer.disconnect();
  },
  methods: {
    initializeObserver() {
      this.observer = new IntersectionObserver(this.handleObserve);
      this.observer.observe(this.$el);
    },
    handleObserve(entries) {
      this.isVisible = entries.filter((entry) => entry.isIntersecting).length;

      for (const entry of entries) {
        this.observer.unobserve(entry.target);
      }
    },
  },
};
</script>
