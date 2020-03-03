<template>
  <div>
    <button @click="text = '321'">321</button>
    <div ref="text">{{text}}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text: "123"
    };
  },
  beforeCreate() {
    console.log('------beforeCreate--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$el);
  },
  created() {
    console.log('------created--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$el);
  },
  beforeMount() {
    console.log('------beforeMount--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$el);
  },
  mounted() {
    console.log('------mounted--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$el);
  },
  beforeUpdate() {
    console.log('------beforeUpdate--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$refs['text']);
  },
  updated() {
    console.log('------updated--------')
    console.log(this.text);
    console.log(this);
    console.log(this.$refs['text']);
  }
}
</script>

<style>

</style>