<template>
  <main class="index-page">
    <SliderComponent
      :breakpoints="[
        {
          media: '(min-width: 320px)',
          items: 2,
        },
        {
          media: '(min-width: 1024px)',
          items: 4,
        },
      ]"
    >
      <template slot="header">
        <h1>Featured</h1>
      </template>
      <component
        :is="'ProductCard'"
        v-for="item in products.slice(0, 14)"
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
  letter-spacing: 0.01em;
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
  padding: 0;
  margin: 0;
  border: 0;
  font-size: inherit;
  color: currentColor;
  background-color: transparent;
  appearance: none;
  cursor: pointer;
  letter-spacing: inherit;
}
</style>
