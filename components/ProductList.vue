<template>
  <section v-if="productsLoaded" class="product-list">
    <header v-if="title" class="product-list-header">
      <h1 class="product-list-title">{{ title }}</h1>
    </header>
    <div class="product-list-container">
      <ProductCard
        v-for="(product, index) in productsLoaded"
        ref="productCard"
        :key="index"
        v-bind="product"
      />
    </div>
    <footer v-if="showLoadMore" class="product-list-footer">
      <LoadMoreButton
        ref="loadMoreButton"
        class="load-more-button"
        @click="
          () => {
            hasLoadedMore = true;
            loadMoreProducts();
          }
        "
      />
    </footer>
  </section>
</template>

<script>
import ProductCard from "./ProductCard.vue";
import LoadMoreButton from "./LoadMoreButton.vue";

export default {
  name: "ProductList",
  components: {
    ProductCard,
    LoadMoreButton,
  },
  props: {
    products: {
      type: Array,
      default: () => [],
    },
    limit: {
      type: Number,
      default: 24,
    },
    title: {
      type: String,
      default: undefined,
    },
  },
  data: () => ({
    observer: undefined,
    page: 1,
    hasLoadedMore: false,
  }),
  computed: {
    productsLoaded() {
      return this.products.slice(0, this.page * this.limit);
    },
    showLoadMore() {
      return this.productsLoaded.length < this.products.length;
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
        rootMargin: "0% 0% 100% 0%",
        threshold: [0, 0.25, 0.5, 0.75, 1],
      });

      this.observer.observe(this.$refs.loadMoreButton.$el);
    },
    handleObserve(entries) {
      for (const entry of entries) {
        if (!this.hasLoadedMore) {
          return;
        }

        if (entry.isIntersecting && entry.intersectionRatio > 0) {
          this.loadMoreProducts();
        }
      }
    },
    loadMoreProducts() {
      this.page++;
    },
  },
};
</script>

<style scoped>
.product-list {
  border-bottom: 1px solid var(--color-black);
}

.product-list-header {
  margin: clamp(1.5rem, 3vw, 2rem);
}

.product-list-title {
  font-size: clamp(2rem, 6vw, 10rem);
}

.product-list-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  column-gap: 1px;
}

.load-more-button {
  width: 100%;
  font-size: clamp(1.4rem, 4vw, 2rem);
}

@media (min-width: 1024px) {
  .product-list-container {
    grid-template-columns: repeat(auto-fill, minmax(24rem, 1fr));
  }
}
</style>
