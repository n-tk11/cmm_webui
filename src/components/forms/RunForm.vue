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
    <input type="text" id="imageUrl" style="width: 300px;" v-model="formData.image_url" />
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
    <button @click="submitForm">Submit</button>
  </div>
</template>

<script>
import { onMounted, ref, defineComponent } from 'vue';
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';
export default defineComponent({
  setup(props, { emit }) {
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
    const submitForm = async () => {
      // Convert newline-separated strings to arrays
      formData.envs = envsText.value.split('\n').filter(env => env.trim() !== '');


      console.log('Form Data:', formData);
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/run/${runWorker.value}/${service.value}`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(formData.value),
        });

        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = 'Service(' + formData.value.container_name + ') is now running on ' + runWorker.value;
          toast.success(msg)
        } else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
        }
      } catch (error) {
        const errorText = 'Error submitting form: ' + error.message;
        console.error('Error submitting form:', error);
        toast.error(errorText);
        // Handle the error as needed
      }
      emit('submit-form', formData.value);
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
    const fetchConfig = async () => {
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/service/${service.value}/config`;
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
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    onMounted(fetchData);
    const isSectionVisible = ref(false);
    const toggleVisibility = () => {
      isSectionVisible.value = !isSectionVisible.value;
    };
    return {
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
    };
  },
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
</style>