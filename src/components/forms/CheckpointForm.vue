<template>
  <div>
    <h2>Checkpoint Configuration Form</h2>
    <label for="workerName">Worker Name:</label>
    <select id="workerName" v-model="chkWorker">
      <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
    </select>
    <label for="containerName">Service Name:</label>
    <select id="containerName" v-model="formData.container_name">
      <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
    </select>
    <br>
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

    <label for="envs">Environment Variables:</label>
    <textarea id="envs" v-model="envsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>

    <button @click="submitForm">Submit</button>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
export default {

  methods: {
    async submitForm() {
      // Convert newline-separated strings to arrays
      this.formData.envs = this.envsText.split('\n').filter(env => env.trim() !== '');

      // Emit an event to inform the parent component about the form submission
      this.$emit('submit-form', this.formData);

      console.log('Form Data:', this.formData);
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/checkpoint/${this.chkWorker}/${this.formData.container_name}`;
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
  },
  setup() {
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
    const submitForm = async () => {
      // Convert newline-separated strings to arrays
      formData.value.envs = envsText.value.split('\n').filter(env => env.trim() !== '');

      // Emit an event to inform the parent component about the form submission

      console.log('Form Data:', formData.value);
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/checkpoint/${chkWorker.value}/${formData.value.container_name}`;
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
      submitForm,
      formData,
      chkWorker,
      envsText,
    };
  },
};
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
