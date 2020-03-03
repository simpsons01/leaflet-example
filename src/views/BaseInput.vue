<template>
  <div>
    <div>
      <input type="text" :value="text1" @input="text1 = $event.target.value" />
      {{text1}}
    </div>
    <div>
      <input type="text" v-model="text2" />
      {{text2}}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text1: "123",
      text2: "123"
    };
  },
  props: ["parentVal"],
  methods: {}
};
</script>

<style>
</style>