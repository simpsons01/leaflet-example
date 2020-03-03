<template>
  <div>
    <label for="age">滿18歲</label>
    <input :checked="checked" id="age" name="age" type="checkbox" />
  </div>
</template>

<script>
export default {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    // this allows using the `value` prop for a different purpose
    value: String,
    // use `checked` as the prop which take the place of `value`
    checked: {
      type: Boolean,
      default: false
    }
  },
};
</script>

<style>
</style>