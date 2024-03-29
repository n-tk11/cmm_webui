<template>
  <div>
    <h2>Checkpoint Configuration Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="chkWorker">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
    </select>
    <label for="containerName">Service Name:</label>
    <select id="containerName" v-model="formData.container_name" @change="fetchConfig">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <br>
    <button @click="toggleVisibility" style="margin-bottom: 5px;">
      <span v-if="!isSectionVisible">▼</span>
      <span v-else>▲</span>
      Advance Config
    </button>
    <div v-if="isSectionVisible" style="padding: 0px;">
      <label for="leaveRunning">Leave Running:</label>
      <input type="checkbox" id="leaveRunning" v-model="formData.leave_running" />

      <label for="imageUrl">Image URL:</label>
      <input type="text" id="imageUrl" v-model="formData.image_url" />

      <label for="passphraseFile">Passphrase File:</label>
      <input type="text" id="passphraseFile" v-model="formData.passphrase_file" />
      <br>
      <label for="preservedPaths">Preserved Paths:</label>
      <input type="text" id="preservedPaths" v-model="formData.preserved_paths" />

      <label for="numShards">Number of Shards:</label>
      <input type="number" min="1" id="numShards" v-model="formData.num_shards" />

      <label for="cpuBudget">CPU Budget:</label>
      <select id="cpuBudget" v-model="formData.cpu_budget">
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
      </select>

      <label for="verbose">Verbose:</label>
      <input type="number" id="verbose" min="1" v-model="formData.verbose" />
      <br>
      <label for="envs">Environment Variables:</label>
      <textarea id="envs" v-model="envsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
    </div>
    <br>

    <button @click="submitForm">Submit</button>
    <LoadingSpinner :isLoading="is_Loading" />
  </div>
</template>

<script>
import { onMounted, ref, defineComponent } from 'vue';
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';
import LoadingSpinner from '../LoadingSpinner.vue';
export default defineComponent({
  conponents: {
    LoadingSpinner,
  },
  setup(props, { emit }) {
    const is_Loading = ref(false);
    const formData = ref({
      leave_running: false,
      image_url: '',
      passphrase_file: '',
      preserved_paths: '',
      num_shards: 3,
      cpu_budget: 'medium',
      verbose: 1,
      envs: [],
    });
    const envsText = ref('');
    const chkWorker = ref('');
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      is_Loading.value = true;
      // Convert newline-separated strings to arrays
      formData.value.envs = envsText.value.split('\n').filter(env => env.trim() !== '');
      // Emit an event to inform the parent component about the form submission
      console.log('Form Data:', formData.value);
      try {
        const url = `${root_url}/checkpoint/${chkWorker.value}/${formData.value.container_name}`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(formData.value),
        });
        is_Loading.value = false;
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service(${formData.value.container_name}) on ${chkWorker.value} is checkpointed`;
          toast.success(msg);
        }
        else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
          // Handle the error as needed
        }
      }
      catch (error) {
        is_Loading.value = false;
        const errorText = await response.text();
        const errorJson = JSON.parse(errorText);
        console.error('Error submitting form:', errorJson.error);
        toast.error(errorJson.error);
      }
      finally {
        is_Loading.value = false;
      }
      emit('submit-form', formData.value);
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
        }
        else {
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
        }
        else {
          throw new Error('Request services failed!');
        }
      }
      catch (error) {
        console.log(error);
      }
    };
    const fetchConfig = async () => {
      try {
        const url = `${root_url}/service/${formData.value.container_name}/config`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          formData.value.leave_running = data.chk_opt.leave_running;
          formData.value.image_url = "";
          formData.value.passphrase_file = data.chk_opt.passphrase_file;
          formData.value.preserved_paths = data.chk_opt.preserved_paths;
          formData.value.num_shards = data.chk_opt.num_shards;
          formData.value.cpu_budget = data.chk_opt.cpu_budget;
          formData.value.verbose = data.chk_opt.verbose;
          envsText.value = data.chk_opt.envs.join('\n');
        }
        else {
          throw new Error('Request services failed!');
        }
      }
      catch (error) {
        console.log(error);
      }
    };
    onMounted(fetchData);
    const isSectionVisible = ref(false);
    const toggleVisibility = () => {
      isSectionVisible.value = !isSectionVisible.value;
    };
    return {
      is_Loading,
      workers,
      services,
      submitForm,
      formData,
      chkWorker,
      envsText,
      fetchConfig,
      isSectionVisible,
      toggleVisibility,
    };
  },
  components: { LoadingSpinner }
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
