<template>
  <div class="circle" :style="{ backgroundColor: color }"></div>
</template>

<script>
export default {
  props: {
    color: {
      type: String,
      default: 'gray', // Default color
    },
  },
};
</script>

<style scoped>
.circle {
  border-radius: 50%;
  width: 20px;
  /* Adjust the size of the circle as needed */
  height: 20px;
  /* Adjust the size of the circle as needed */
  display: inline-block;
  /* Add margin for spacing */
  margin-right: 5px;
}
</style>