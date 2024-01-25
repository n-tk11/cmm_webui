<template>
  <div>
    <h2>Delete a Worker</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="workerName">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
    </select>
    <br>
    <LoadingSpinner :isLoading="is_Loading" />
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
    const workerName = ref('');
    const workers = ref([]);
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      is_Loading.value = true;
      try {
        const url = `${root_url}/worker/${workerName.value}`;
        const response = await fetch(url, {
          method: 'DELETE',
        });
        is_Loading.value = false;
        console.log(workerName.value);
        if (response.status === 204) {
          console.log('Form submitted successfully');
          const msg = `Worker ${workerName.value} is deleted`;
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
      emit('submit-form');
    };

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
      } catch (error) {
        console.error('Error fetching data:', error);
        const errorText = 'Error fetching data: ' + error.message;
        toast.error(errorText);
      }
    };
    onMounted(() => {
      fetchData();
    });

    return {
      is_Loading,
      workers,
      workerName,
      fetchData,
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
