<template>
  <figure class="picture" ref="image">
    <img
      :src="src"
      :alt="alt"
      :loading="lazyload ? 'lazy' : 'eager'"
      @load="imageLoaded"
    />
  </figure>
</template>

<script>
export default {
  name: "Picture",
  props: {
    src: {
      type: String,
      required: true,
    },
    alt: {
      type: String,
      default: "",
    },
    lazyload: {
      type: Boolean,
      default: true,
    },
    delay: {
      type: Number,
      default: 50,
    },
  },
  methods: {
    imageLoaded() {
      setTimeout(() => this.$refs.image.classList.add("is-loaded"), this.delay);
    },
  },
};
</script>

<style scoped>
.picture img {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transform: scale(1.05);
  transition-property: opacity, transform;
  transition-duration: 400ms, 800ms;
  transition-timing-function: cubic-bezier(0, 0.4, 0.6, 1);
}

.picture.is-loaded img {
  opacity: 1;
  transform: scale(1);
}
</style>
