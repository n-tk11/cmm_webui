<template>
  <div>
    <h2>Start Service Container Configuration Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="startWorker">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
      <!-- Add more options as needed -->
    </select>
    <label for="containerName">Service(container) Name:</label>
    <select id="containerName" v-model="formData.container_name">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <br>
    <label for="appPort">App Port:</label>
    <input type="text" id="appPort" v-model="formData.app_port" />
    <br>
    <label for="envs">Environment Variables:</label>
    <textarea id="envs" v-model="envsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>

    <div v-for="(mount, index) in formData.mounts" :key="index">
      <label for="mountType">Mount Type:</label>
      <select v-model="mount.type">
        <option value="bind">Bind</option>
        <option value="volume">Volume</option>
        <!-- Add more options as needed -->
      </select>

      <label for="mountSource">Mount Source:</label>
      <input type="text" v-model="mount.source" />

      <label for="mountTarget">Mount Target:</label>
      <input type="text" v-model="mount.target" />

      <label for="mountReadonly">Mount Readonly:</label>
      <input type="checkbox" v-model="mount.readonly" />

      <button @click="removeMount(index)">Remove Mount</button>
    </div>
    <button @click="addMount">Add Mount</button>
    <br>
    <label for="caps">Capabilities:</label>
    <textarea id="caps" v-model="capsText" placeholder="CAP1&#10;CAP2"></textarea>

    <button @click="submitForm">Submit</button>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';

/*
export default {
  setup() {
    onMounted(fetchData);
  },
  data() {
    return {
      formData: {
        container_name: '',
        image: '',
        app_port: '',
        envs: [],
        mounts: [
          {
            type: 'bind',
            source: '',
            target: '',
            readonly: false,
          },
        ],
        caps: [],
      },
      envsText: '',
      capsText: '',
      workers: [],
      services: [],
      startWorker: '',
    };
  },
  methods: {

    async submitForm() {
      // Convert newline-separated strings to arrays
      this.formData.envs = this.envsText.split('\n').filter(env => env.trim() !== '');
      this.formData.caps = this.capsText.split('\n').filter(cap => cap.trim() !== '');
      const url = `http://localhost:8080/cm_manager/v1.0/start/${this.startWorker}/${this.formData.container_name}`;
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(formData.value),
        });

        if (response.ok) {
          console.log('Form submitted successfully');
          // Optionally, close the form or perform other actions upon successful submission
        } else {
          console.error('Error submitting form:', response.statusText);
          // Handle the error as needed
        }
      } catch (error) {
        console.error('Error submitting form:', error);
        // Handle the error as needed
      }
    },
    addMount() {
      this.formData.mounts.push({
        type: 'bind',
        source: '',
        target: '',
        readonly: false,
      });
    },
    removeMount(index) {
      this.formData.mounts.splice(index, 1);
    },
  },
};
*/
export default {
  setup() {
    const formData = ref({
      container_name: '',
      image: '',
      app_port: '',
      envs: [],
      mounts: [
        {
          type: 'bind',
          source: '',
          target: '',
          readonly: false,
        },
      ],
      caps: [],
    });

    const envsText = ref('');
    const capsText = ref('');
    const workers = ref([]);
    const services = ref([]);
    const startWorker = ref('');

    const fetchData = async () => {
      try {
        const response = await fetch('http://localhost:8080/cm_manager/v1.0/worker');
        if (response.ok) {
          const data = await response.json();
          workers.value = Array.from(new Set(data.map(item => item.id)));
        }
        else {
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

    const submitForm = async () => {
      // Convert newline-separated strings to arrays
      formData.value.envs = envsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.caps = capsText.value.split('\n').filter(cap => cap.trim() !== '');
      const url = `http://localhost:8080/cm_manager/v1.0/start/${startWorker.value}/${formData.value.container_name}`;
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(formData.value),
        });
        $emit('submit-form', formData.value)
        if (response.ok) {
          console.log('Form submitted successfully');
          // Optionally, close the form or perform other actions upon successful submission
        } else {
          console.error('Error submitting form:', response.statusText);
          // Handle the error as needed
        }
      } catch (error) {
        console.error('Error submitting form:', error);
        // Handle the error as needed
      }
    };
    onMounted(fetchData);
    const addMount = () => {
      formData.value.mounts.push({
        type: 'bind',
        source: '',
        target: '',
        readonly: false,
      });
    };

    const removeMount = (index) => {
      formData.value.mounts.splice(index, 1);
    };

    return {
      formData,
      envsText,
      capsText,
      workers,
      services,
      startWorker,
      submitForm,
      addMount,
      removeMount,
    };
  },
};
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
</style>

