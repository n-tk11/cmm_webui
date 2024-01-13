<template>
  <div>
    <h2>Add Service Form</h2>
    <label for="serviceName">Service Name:</label>
    <input type="text" id="serviceName" v-model="service.serviceName">
    <label for="serviceImage">Image Name and tag:</label>
    <input type="text" id="serviceImage" v-model="service.serviceImage" placeholder="ffdev:10">
    <br>
    <button @click="submitForm">Submit</button>
  </div>
</template>

<script>
import { onMounted, ref, defineComponent } from 'vue';
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';
export default defineComponent({
  setup(props, { emit }) {
    const service = ref({
      serviceName: '',
      serviceImage: '',
    });
    const submitForm = async () => {
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/service`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(service.value),
        });
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service ${service.value.serviceName} with image ${service.value.serviceImage} is added`;
          toast.success(msg)
        } else {
          console.error('Error submitting form:', response.statusText);
          const errorText = await response.text();
          errorText = response.statusText + ' ' + errorText;
          toast.error(errorText);
          // Handle the error as needed
        }
      } catch (error) {
        console.error('Error submitting form:', error);
        const errorText = 'Error submitting form: ' + error.message;
        toast.error(errorText);
        // Handle the error as needed
      }
      emit('submit-form');
    };



    return {
      service,
      submitForm,
    };
  },
});
</script>

<style scoped>
/* Your styles for the ImageForm component go here */
div {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
}

textarea {
  width: 100%;
  height: 80px;
}
</style>
