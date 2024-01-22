<template>
  <nav class="side-navbar">
    <h1>CMM</h1>
    <ul>
      <li @click="openForm('start')">Start</li>
      <li @click="openForm('run')">Run</li>
      <li @click="openForm('checkpoint')">Checkpoint</li>
      <li @click="openForm('migrate')">Migrate</li>
      <li @click="openForm('stop')">
        Stop</li>
      <li @click="openForm('remove')">Remove</li>
    </ul>
  </nav>
</template>

<script>
export default {
  methods: {
    openForm(route) {
      this.$emit('open-form', route);
    },
  },
};
</script>

<style scoped>
/* Your styles for the side navbar go here */
.side-navbar {
  width: 200px;
  background-color: #333;
  color: white;
  padding: 20px;
  overflow: auto;
}

.side-navbar ul {
  list-style: none;
  padding: 0;
}

.side-navbar li {
  margin-bottom: 10px;
  cursor: pointer;
  font-size: 20px;
}

.side-navbar a {
  text-decoration: none;
  color: white;
}
</style>
