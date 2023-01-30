<template>
  <main class="index-page">
    <SliderComponent
      gap="1px"
      :breakpoints="[
        {
          media: '(min-width: 320px)',
          width: '50vw',
          peek: '2vw',
        },
        {
          media: '(min-width: 1024px)',
          width: '25vw',
          peek: '1vw',
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
