<template>
  <div>S1
     <button @click="count += 1">+1</button>
     {{count}}
  </div>
</template>

<script>
export default {
   name:'S1',
   data() {
     return {
        type:'S1',
        count:0
     }
   }
}
</script>

<style>

</style>