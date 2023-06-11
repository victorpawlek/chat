<template>
  <div class="home">
    <h2 class="text-center my-3">Home</h2>
    <MessageView serverUrl="ws://localhost:3000" />
  </div>
</template>

<script>
import MessageView from '@/components/MessageView.vue';

export default {
  name: 'Home',
  components: {
    MessageView
  },
};
</script>
