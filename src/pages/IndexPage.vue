<template>
  <SignIn></SignIn>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import SignIn from 'layouts/SignIn.vue';
export default defineComponent({
  name: 'IndexPage',
  components: {
    SignIn,
  },
  setup() {
    return {};
  },
});
</script>
