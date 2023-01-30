<template>
  <figure ref="image" class="image">
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
  name: "ImageComponent",
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
