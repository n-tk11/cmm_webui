<template>
  <div>
    <h2>Stop Form</h2>
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
import LoadingSpinner from '../LoadingSpinner.vue';
export default defineComponent({
  components: {
    LoadingSpinner,
  },
  setup(props, { emit }) {
    const is_Loading = ref(false);
    const stopWorker = ref('');
    const service = ref('');
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      is_Loading.value = true;
      try {
        const url = `${root_url}/stop/${stopWorker.value}/${service.value}`;
        const response = await fetch(url, {
          method: 'POST',
        });
        is_Loading.value = false;
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service(${service.value}) on ${stopWorker.value} is stopped`;
          toast.success(msg)
        } else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
        }
      } catch (error) {
        is_Loading.value = false;
        console.error('Error submitting form:', error);
        const errorText = 'Error submitting form: ' + error.message;
        toast.error(errorText);
        // Handle the error as needed
      } finally {
        is_Loading.value = false;
      }
      const formData = {
        worker_name: stopWorker.value,
        service_name: service.value,
      };
      emit('submit-form', formData);
    };
    const workers = ref([]);
    const services = ref([]);
    const fetchData = async () => {
      try {
        const url = `${root_url}/worker`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          if (data === null || data.length === 0) {
            workers.value = [];
            return;
          }
          workers.value = data.sort((a, b) => a.id.localeCompare(b.id)).map(item => item.id);
        } else {
          throw new Error('Request workers failed!');
        }
        const url2 = `${root_url}/service`;
        const response2 = await fetch(url2);
        if (response2.ok) {
          const data2 = await response2.json();
          if (data2 === null || data2.length === 0) {
            services.value = [];
            return;
          }
          services.value = data2.sort((a, b) => a.name.localeCompare(b.name)).map(item => item.name);
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    onMounted(fetchData);

    return {
      is_Loading,
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
