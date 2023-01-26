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

<style scoped>
.product-card {
  font-size: 0.875rem;
  box-shadow: 0 0 0 1px #000;
  background-color: #ffffff;
}

.product-card-image {
  position: relative;
  background-color: hsla(0, 0%, 0%, 0.05);
  overflow: hidden;
  aspect-ratio: 5 / 7;
}

.product-card-content {
  padding: 1.2em;
}

.product-card-name {
  font-size: 1em;
  margin-bottom: 0.3em;
}

.product-card-price {
  margin-bottom: 0;
}
</style>
