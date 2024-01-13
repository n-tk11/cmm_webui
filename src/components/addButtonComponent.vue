
<template>
  <!-- <div class="add-button">
    <div class="text-wrapper" @click="openForm('addWorker')">Add a worker</div>
    <Add24Px class="content-add" />
  </div> -->
  <button @click="openForm('addWorker')">Add a worker</button>
</template>

<script>


export default {
  name: "AddButton",

  methods: {
    openForm(route) {
      this.$emit('open-form', route);
    },
  },

};
</script>

<style>
.add-button {
  background-color: #7df08f;
  border-radius: 10px;
  height: 48px;
  overflow: hidden;
  position: relative;
  width: 170px;
}

.add-button .text-wrapper {
  color: #000000;
  font-family: "Inter-Regular", Helvetica;
  font-size: 20px;
  font-weight: 400;
  left: 31px;
  letter-spacing: 0;
  line-height: normal;
  position: absolute;
  top: 11px;
  white-space: nowrap;
}

.add-button .content-add {
  height: 24px !important;
  left: 7px !important;
  position: absolute !important;
  top: 12px !important;
  width: 24px !important;
}
</style>