<template>
  <div>
    <h2>Add Service Form</h2>
    <label for="serviceName">Service Name:</label>
    <input type="text" id="serviceName" v-model="service.name">
    <label for="serviceImage">Image Name and tag:</label>
    <input type="text" id="serviceImage" v-model="service.image" placeholder="ffdev:10">
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
      name: '',
      image: '',
    });
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      try {
        const url = `${root_url}/service`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(service.value),
        });
        console.log(service.value);
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service ${service.value.serviceName} with image ${service.value.serviceImage} is added`;
          toast.success(msg)
        } else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
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

select {
  margin-left: 3px;
  margin-right: 15px;
}

input {
  margin-left: 3px;
  margin-right: 15px;
}

div {
  line-height: 2;
}
</style>
