<template>
   <div>
     <s1 ref="s1"/>
     <s2 ref="s2"/>
   </div>
</template>

<script>
function getS1() {
  return import(/* webpackChunkName: "S1" */ "./S1").then(
    ({ default: S1 }) => {
      return S1;
    }
  );
}
function getS2() {
  return import(/* webpackChunkName: "S2" */ "./S2").then(
    ({ default: S2 }) => {
      return S2;
    }
  );
}
// function setTime(time) {
//   return new Promise(resolve => {
//     setTimeout(() => resolve(time), time);
//   });
// }


export default {
components: {
    S1:getS1,
    S2:getS2,
  },
  methods: {
  },
  async mounted() {
    console.log(this.$options.components)
    getS1().then(()=>{
       console.log(this.$options.components.S1)
    })  
  }
}
</script>

<style>

</style>