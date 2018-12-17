<template>
  <th
    :style="{width: vuetable.scrollBarWidth}"
    class="vuetable-th-gutter"
    v-if="vuetable.scrollBarWidth === '0px'"
  ></th>
</template>

<script>
export default {
  name: 'vuetable-th-gutter',

  computed: {
    vuetable() {
      return this.$parent
    }
  }
}
</script>

<style>
  .vuetable-th-gutter {
    padding: 0 !important;
    border-left: none  !important;
    border-right: none  !important;
  }
</style>
