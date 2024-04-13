<script>
export default {
  props: { contributors: Map },
  data: () => ({}),
  computed: {
    topContributorInfo: {
      get() {
        return this.getTopContributor(this.contributors);
      },
    },
    topContributor() {
      return (
        this.topContributorInfo[0] +
        " added " +
        this.topContributorInfo[1] +
        " commits"
      );
    },
  },
  methods: {
    getTopContributor() {
      let maxKey = null;
      let maxValue = -Infinity;
      for (let [key, value] of this.contributors) {
        if (value > maxValue) {
          maxKey = key;
          maxValue = value
        }
      }

      return [maxKey, maxValue];
    },
  },
};
</script>

<template>
  <div class="centered">
    <p class="sub-title">🎉 Top Contributor: {{ topContributor }}</p>
  </div>
</template>

<style scoped>
.sub-title {
  font-size: 1.5rem;
}
</style>
