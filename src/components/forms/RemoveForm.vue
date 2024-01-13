<template>
  <div>
    <h2>Remove Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="stopWorker">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
    </select>
    <label for="containerName">Service Name:</label>
    <select id="containerName" v-model="service">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
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
    const stopWorker = ref('');
    const service = ref('');
    const submitForm = async () => {
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/remove/${stopWorker.value}/${service.value}`;
        const response = await fetch(url, {
          method: 'POST',
        });
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service(${service.value}) on ${stopWorker.value} is removed`;
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
    const workers = ref([]);
    const services = ref([]);
    const fetchData = async () => {
      try {
        const response = await fetch('http://localhost:8080/cm_manager/v1.0/worker');
        if (response.ok) {
          const data = await response.json();
          workers.value = Array.from(new Set(data.map(item => item.id)));
        } else {
          throw new Error('Request workers failed!');
        }
        const response2 = await fetch('http://localhost:8080/cm_manager/v1.0/service');
        if (response2.ok) {
          const data2 = await response2.json();
          services.value = Array.from(new Set(data2.map(item => item.name)));
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    onMounted(fetchData);

    return {
      workers,
      services,
      stopWorker,
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
