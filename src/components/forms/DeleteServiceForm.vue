<template>
  <div>
    <h2>Delete a Service</h2>
    <label for="serviceName">Service Name:</label>
    <select id="serviceName" v-model="serviceName">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <label for="delChk">Delete all related checkpoint files:</label>
    <input type="checkbox" id="delChk" v-model="delChk" />
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
    const serviceName = ref('');
    const services = ref([]);
    const delChk = ref(false);
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      try {
        const url = `${root_url}/service/${serviceName.value}?delChk=${delChk.value}`;
        console.log(url);
        const response = await fetch(url, {
          method: 'DELETE',
        });
        console.log(serviceName.value);
        if (response.status === 204) {
          console.log('Form submitted successfully');
          const msg = `Service ${serviceName.value} is deleted`;
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

    const fetchData = async () => {
      try {
        const url = `${root_url}/service`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          if (data === null || data.length === 0) {
            services.value = [];
            return;
          }
          services.value = data.sort((a, b) => a.name.localeCompare(b.name)).map(item => item.name);
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
      delChk,
      services,
      serviceName,
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
