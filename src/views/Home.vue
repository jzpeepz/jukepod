<template>
  <div class="home">
    <Jukebox/>
  </div>
</template>

<script>
// @ is an alias to /src
import Jukebox from '@/components/Jukebox.vue';

export default {
  name: 'Home',
  components: {
    Jukebox,
  },
};
</script>
