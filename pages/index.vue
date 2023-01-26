<template>
  <main class="index-page">
    <SliderComponent>
      <template slot="header">
        <h1>Featured products</h1>
      </template>
      <component
        :is="ProductCard"
        v-for="item in products.slice(0, 10)"
        :key="item.name"
        v-bind="item"
        slot="content"
      />
    </SliderComponent>
    <ProductList :products="products" title="All products" />
  </main>
</template>

<script>
import Vue from "vue";
import SliderComponent from "~/components/SliderComponent.vue";
import ProductCard from "~/components/ProductCard.vue";
import ProductList from "~/components/ProductList.vue";

const products = [...Array(200)].map((_, index) => {
  const currentProductNumber = index + 1;

  return {
    image: `https://source.unsplash.com/random/${currentProductNumber}`,
    name: `Product ${currentProductNumber}`,
    price: `100 SEK`,
  };
});

export default Vue.extend({
  name: "IndexPage",
  components: {
    SliderComponent,
    ProductCard,
    ProductList,
  },
  data: () => ({
    products,
  }),
});
</script>

<style>
body {
  margin: 0;
  font-family: -apple-system, system-ui, sans-serif;
  line-height: 1.5;
}

img {
  max-width: 100%;
  height: auto;
}

figure {
  margin: 0;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  font-weight: 400;
  margin-top: 0;
  margin-bottom: 1rem;
  line-height: 1.2;
}

p {
  margin-top: 0;
  margin-bottom: 1rem;
}

button {
  font-size: inherit;
  color: currentColor;
  background-color: transparent;
  border: 0;
  appearance: none;
  cursor: pointer;
}
</style>
