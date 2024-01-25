<template>
  <div class="form-overlay" @click="clickClose">
    <div class="form-container">
      <button @click="closeForm">Close Form</button>
      <component :is="getFormComponent" @submit-form="submitForm" @submit-form-success="handleFormSubmit"></component>
      <!-- <button @click="submitForm">Submit</button> -->
    </div>
  </div>
</template>

<script>
import StartForm from './forms/StartForm.vue';
import RunForm from './forms/RunForm.vue';
import CheckpointForm from './forms/CheckpointForm.vue';
import MigrateForm from './forms/MigrateForm.vue';
import StopForm from './forms/StopForm.vue';
import RemoveForm from './forms/RemoveForm.vue';
import AddWorkerForm from './forms/AddWorkerForm.vue';
import AddServiceForm from './forms/AddServiceForm.vue';
import DeleteWorkerForm from './forms/DeleteWorkerForm.vue';
import DeleteServiceForm from './forms/DeleteServiceForm.vue';
import LoadingSpinner from './LoadingSpinner.vue';
export default {
  props: {
    formType: String,
  },
  methods: {
    clickClose(event) {
      // Check if the clicked element is the overlay itself (not the modal content)
      if (event.target.classList.contains('form-overlay')) {
        // Emit an event to close the overlay
        this.$emit('close-form');
      }
    },
    closeForm() {
      this.$emit('close-form');
    },
    submitForm(formData) {
      // Logic to handle form submission based on the form type
      console.log(`Form submitted for ${this.formType} with data:`, formData);
      // Close the form after submission
      this.$emit('form-submitted');
      this.closeForm();
    },
    handleFormSubmit() {
      // Close the form overlay after successful form submission
      this.closeForm();
    },
  },
  computed: {
    getFormComponent() {
      switch (this.formType) {
        case 'start':
          return StartForm;
        case 'run':
          return RunForm;
        case 'checkpoint':
          return CheckpointForm;
        case 'migrate':
          return MigrateForm;
        case 'stop':
          return StopForm;
        case 'remove':
          return RemoveForm;
        case 'addWorker':
          return AddWorkerForm;
        case 'addService':
          return AddServiceForm;
        case 'deleteWorker':
          return DeleteWorkerForm;
        case 'deleteService':
          return DeleteServiceForm;
        default:
          return null; // Default to no form
      }
    },
  },
};
</script>

<style scoped>
/* Your styles for the form overlay and container go here */
.form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.form-container {
  background: white;
  padding: 20px;
  border-radius: 8px;
  position: relative;
}

.form-container button {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}
</style>