<template>
  <Search msg="I'm you father Luke" />
</template>

<script lang="ts">
import Vue from 'vue'
import Search from '~/components/Search.vue'

export default Vue.extend({
  components: { Search },
  name: 'IndexPage'
})
</script>
