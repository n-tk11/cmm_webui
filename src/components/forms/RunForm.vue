<template>
  <div>
    <h2>Run Configuration Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="runWorker">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
    </select>
    <label for="containerName">Service Name:</label>
    <select id="containerName" v-model="service" @change="fetchConfig">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <br>
    <label for="appArgs">App Arguments:</label>
    <input type="text" id="appArgs" style="width: 300px;" v-model="formData.app_args" />
    <br>
    <label for="imageUrl">Image URL:</label>
    <select id="imageUrl" v-model="formData.image_url">
      <option value="">Start From Scratch</option>
      <option v-for="url in servicesChkF" :key="url" :value="url">{{ formatDate(url) }}</option>
      <!-- <input type="text" id="imageUrl" style="width: 300px;" v-model="formData.image_url" /> -->
    </select>
    <br>
    <button @click="toggleVisibility" style="margin-bottom: 5px;">
      <span v-if="!isSectionVisible">▼</span>
      <span v-else>▲</span>
      Advance Config
    </button>

    <div v-if="isSectionVisible" style="padding: 0px;">
      <label for="onAppReady">On App Ready Script:</label>
      <input type="text" id="onAppReady" style="width: 300px;" v-model="formData.on_app_ready" />
      <br>
      <label for="passphraseFile">Passphrase File:</label>
      <input type="text" id="passphraseFile" style="width: 300px;" v-model="formData.passphrase_file" />
      <br>
      <label for="preservedPaths">Preserved Paths:</label>
      <input type="text" id="preservedPaths" style="width: 300px;" v-model="formData.preserved_paths" />
      <br>
      <label for="noRestore">No Restore:</label>
      <input type="checkbox" id="noRestore" v-model="formData.no_restore" />

      <label for="allowBadImage">Allow Bad Image:</label>
      <input type="checkbox" id="allowBadImage" v-model="formData.allow_bad_image" />

      <label for="leaveStopped">Leave Stopped:</label>
      <input type="checkbox" id="leaveStopped" v-model="formData.leave_stopped" />

      <label for="verbose">Verbose:</label>
      <select id="verbose" v-model="formData.verbose">
        <option type="number" value=1>1</option>
        <option type="number" value=2>2</option>
        <option type="number" value=3>3</option>
      </select>
      <br>
      <label for="envs">Environment Variables:</label>
      <textarea id="envs" v-model="envsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
    </div>
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
    const formData = ref({
      app_args: '',
      image_url: '',
      on_app_ready: '',
      passphrase_file: '',
      preserved_paths: '',
      no_restore: false,
      allow_bad_image: false,
      leave_stopped: false,
      verbose: 1,
      envs: [],
    });
    const envsText = ref('');
    const runWorker = ref('');
    const service = ref('');
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      is_Loading.value = true;
      // Convert newline-separated strings to arrays
      formData.envs = envsText.value.split('\n').filter(env => env.trim() !== '');
      console.log('Form Data:', formData);
      try {
        const url = `${root_url}/run/${runWorker.value}/${service.value}`;
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
          const msg = 'Service(' + service.value + ') is now running on ' + runWorker.value;
          toast.success(msg);
        }
        else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
        }
      }
      catch (error) {
        is_Loading.value = false;
        const errorText = 'Error submitting form: ' + error.message;
        console.error('Error submitting form:', error);
        toast.error(errorText);
        // Handle the error as needed
      } finally {
        is_Loading.value = false;
      }
      emit('submit-form', formData.value);
    };
    const workers = ref([]);
    const services = ref([]);
    const servicesChkF = ref([]);
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
        const url = `${root_url}/service/${service.value}/config`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          formData.value.app_args = data.run_opt.app_args;
          formData.value.image_url = data.run_opt.image_url;
          formData.value.on_app_ready = data.run_opt.on_app_ready;
          formData.value.passphrase_file = data.run_opt.passphrase_file;
          formData.value.preserved_paths = data.run_opt.preserved_paths;
          formData.value.no_restore = data.run_opt.no_restore;
          formData.value.allow_bad_image = data.run_opt.allow_bad_image;
          formData.value.leave_stopped = data.run_opt.leave_stopped;
          formData.value.verbose = data.run_opt.verbose;
          envsText.value = data.run_opt.envs.join('\n');
        }
        else {
          throw new Error('Request services failed!');
        }
        const url2 = `${root_url}/service/${service.value}`;
        const response2 = await fetch(url2);
        if (response2.ok) {
          const data2 = await response2.json();
          servicesChkF.value = data2.chk_files.reverse();
        }
        else {
          throw new Error('Request services failed!');
        }
      }
      catch (error) {
        console.log(error);
      }
    };
    const formatDate = (url) => {
      const [_, worker, time] = url.split('_');
      const localTime = new Date(time).toLocaleString('en-Gb');
      return `${localTime}@${worker}`;
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
      formData,
      runWorker,
      envsText,
      submitForm,
      fetchConfig,
      service,
      toggleVisibility,
      isSectionVisible,
      servicesChkF,
      formatDate,
    };
  },
  components: { LoadingSpinner }
});
</script>

<style scoped>
/* Your styles for the AppForm component go here */
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