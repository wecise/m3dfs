<template>

  <IndexView :root="{parent:'/', fullname:'/'}"></IndexView>
  
</template>

<script>
import IndexView from './dfs/index.vue';

export default {
  components:{
    IndexView
  }
}
</script>
