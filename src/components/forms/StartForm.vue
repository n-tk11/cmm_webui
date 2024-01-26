<template>
  <div>
    <h2>Start Service Container Configuration Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="startWorker" style="margin-right: 5px;">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
      <!-- Add more options as needed -->
    </select>
    <label for="containerName">Service(container) Name:</label>
    <select id="containerName" v-model="formData.container_name" @change="fetchConfig">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <br>
    <label for="appPort">App Port:</label>
    <textarea id="appPort" v-model="app_ports" placeholder="8080:8080&#10;7777:7777"></textarea>
    <br>
    <div v-for="(mount, index) in formData.mounts" :key="index" style="padding: 5px;">
      <label for="mountType">Mount Type:</label>
      <select v-model="mount.Type">
        <option value="bind">Bind</option>
        <option value="volume">Volume</option>
        <!-- Add more options as needed -->
      </select>

      <label for="mountSource">Mount Source:</label>
      <input type="text" v-model="mount.Source" />

      <label for="mountTarget">Mount Target:</label>
      <input type="text" v-model="mount.Target" />

      <label for="mountReadonly">Mount Readonly:</label>
      <input type="checkbox" v-model="mount.ReadOnly" />

      <button @click="removeMount(index)">Remove Mount</button>
    </div>
    <button @click="addMount" style="margin: 5px;">Add Mount</button>
    <br>
    <button @click="toggleVisibility" style="margin-bottom: 5px;">
      <span v-if="!isSectionVisible">▼</span>
      <span v-else>▲</span>
      Advance Config
    </button>
    <div v-if="isSectionVisible" style="padding: 0px;">
      <label for="envs">Environment Variables:</label>
      <textarea id="envs" v-model="envsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
      <br>
      <label for="caps">Capabilities:</label>
      <textarea id="caps" v-model="capsText" placeholder="CAP1&#10;CAP2"></textarea>
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
      container_name: '',
      image: '',
      app_ports: [],
      envs: [],
      mounts: [
        {
          Type: 'bind',
          Source: '',
          Target: '',
          ReadOnly: false,
        },
      ],
      caps: [],
    });
    const app_ports = ref('');
    const envsText = ref('');
    const capsText = ref('');
    const workers = ref([]);
    const services = ref([]);
    const startWorker = ref('');
    const root_url = import.meta.env.VITE_API_URL;
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
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    const fetchConfig = async () => {
      try {
        const url = `${root_url}/service/${formData.value.container_name}/config`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          formData.value.app_ports = data.start_opt.app_ports.join('\n');
          formData.value.mounts = data.start_opt.mounts;
          envsText.value = data.start_opt.envs.join('\n');
          capsText.value = data.start_opt.caps.join('\n');
          console.log(data.start_opt.mounts);
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    const submitForm = async () => {
      // Convert newline-separated strings to arrays
      is_Loading.value = true;
      formData.value.app_ports = app_ports.value.split('\n').filter(port => port.trim() !== '');
      formData.value.envs = envsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.caps = capsText.value.split('\n').filter(cap => cap.trim() !== '');
      const url = `${root_url}/start/${startWorker.value}/${formData.value.container_name}`;
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(formData.value),
        });
        is_Loading.value = false;
        emit('submit-form', formData.value);
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Service(${formData.value.container_name})'s container is now started on ${startWorker.value}`;
          toast.success("Start Service's Container Successfully!");
        } else {
          const errorText = await response.text();
          const errorJson = JSON.parse(errorText);
          console.error('Error submitting form:', errorJson.error);
          toast.error(errorJson.error);
        }
      } catch (error) {
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
    onMounted(fetchData);
    const addMount = () => {
      formData.value.mounts.push({
        Type: 'bind',
        Source: '',
        Target: '',
        ReadOnly: false,
      });
    };

    const removeMount = (index) => {
      formData.value.mounts.splice(index, 1);
    };
    const isSectionVisible = ref(false);
    const toggleVisibility = () => {
      isSectionVisible.value = !isSectionVisible.value;
    };
    return {
      is_Loading,
      formData,
      app_ports,
      envsText,
      capsText,
      workers,
      services,
      startWorker,
      submitForm,
      addMount,
      removeMount,
      fetchConfig,
      isSectionVisible,
      toggleVisibility,
    };
  },
});
</script>

<style scoped>
/* Your styles for the DockerForm component go here */
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

