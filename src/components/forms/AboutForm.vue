<template>
  <div>
    <h2>Home Form</h2>
    <label for="firstName">First Name:</label>
    <input type="text" id="firstName" v-model="formData.firstName" />

    <label for="lastName">Last Name:</label>
    <input type="text" id="lastName" v-model="formData.lastName" />

    <button @click="submitForm">Submit</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      formData: {
        firstName: '',
        lastName: '',
      },
    };
  },
  methods: {
    submitForm() {
      // Make an HTTP POST request to the API endpoint
      axios
        .post('https://example.com/api/form', this.formData)
        .then(response => {
          console.log('Form submitted successfully:', response.data);
          // Optionally, emit an event to inform the parent component
          this.$emit('submit-form-success', response.data);
        })
        .catch(error => {
          console.error('Error submitting form:', error);
          // Optionally, emit an event to inform the parent component about the error
          this.$emit('submit-form-error', error);
        });
    },
  },
};
</script>

<style scoped>
/* Your styles for the HomeForm component go here */
div {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
}
</style>