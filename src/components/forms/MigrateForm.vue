<template>
  <div>
    <h2>Migrate Configuration Form</h2>
    <div style="padding-top: 0px; margin-left: 0px; width:40%; display: table;">
      <div style="margin-left: 0px; display: table-row">
        <div style="display: table-cell;">
          <label for="sourceName">Source Worker:</label>
          <select id="sourceName" v-model="srcWorker">
            <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
          </select>
        </div>
        <div style="padding: 0px;display: table-cell;"><img class="arrow" alt="Arrow"
            src="../../assets/right-arrow.svg" /></div>
        <div style="display: table-cell;">
          <label for="destName">Destination Worker:</label>
          <select id="destName" v-model="destWorker">
            <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
          </select>
        </div>
      </div>
    </div>
    <div style="padding: 0%; margin-left: 18px;">
      <label for="serviceName">Service:</label>
      <select id="serviceName" v-model="formData.start.container_name">
        source
        <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
      </select>
    </div>

    <!-- Checkpoint Form -->
    <h3>Checkpoint Configuration</h3>
    <label for="leaveRunning">Leave Running:</label>
    <input type="checkbox" id="leaveRunning" v-model="formData.checkpoint.leave_running" />

    <label for="imageUrl">Image URL:</label>
    <input type="text" id="imageUrl" v-model="formData.checkpoint.image_url" />

    <label for="passphraseFile">Passphrase File:</label>
    <input type="text" id="passphraseFile" v-model="formData.checkpoint.passphrase_file" />
    <br>
    <label for="preservedPaths">Preserved Paths:</label>
    <input type="text" id="preservedPaths" v-model="formData.checkpoint.preserved_paths" />

    <label for="numShards">Number of Shards:</label>
    <input type="number" min="1" id="numShards" v-model="formData.checkpoint.num_shards" />

    <label for="cpuBudget">CPU Budget:</label>
    <select id="cpuBudget" v-model="formData.checkpoint.cpu_budget">
      <option value="low">Low</option>
      <option value="medium">Medium</option>
      <option value="high">High</option>
    </select>

    <label for="verbose">Verbose:</label>
    <input type="number" id="verbose" min="1" v-model="formData.checkpoint.verbose" />

    <label for="envs">Environment Variables:</label>
    <textarea id="envs" v-model="chkEnvsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
    <br>
    <!-- Run Form -->
    <h3>Run Configuration</h3>
    <label for="appArgs">App Arguments:</label>
    <input type="text" id="appArgs" style="width: 300px;" v-model="formData.run.app_args" />
    <br>
    <label for="imageUrl">Image URL:</label>
    <input type="text" id="imageUrl" style="width: 300px;" v-model="formData.run.image_url" />
    <br>
    <label for="onAppReady">On App Ready Script:</label>
    <input type="text" id="onAppReady" style="width: 300px;" v-model="formData.run.on_app_ready" />
    <br>
    <label for="passphraseFile">Passphrase File:</label>
    <input type="text" id="passphraseFile" style="width: 300px;" v-model="formData.run.passphrase_file" />
    <br>
    <label for="preservedPaths">Preserved Paths:</label>
    <input type="text" id="preservedPaths" style="width: 300px;" v-model="formData.run.preserved_paths" />
    <br>
    <label for="noRestore">No Restore:</label>
    <input type="checkbox" id="noRestore" v-model="formData.run.no_restore" />

    <label for="allowBadImage">Allow Bad Image:</label>
    <input type="checkbox" id="allowBadImage" v-model="formData.run.allow_bad_image" />

    <label for="leaveStopped">Leave Stopped:</label>
    <input type="checkbox" id="leaveStopped" v-model="formData.run.leave_stopped" />

    <label for="verbose">Verbose:</label>
    <select id="verbose" v-model="formData.run.verbose">
      <option type="number" value=1>1</option>
      <option type="number" value=2>2</option>
      <option type="number" value=3>3</option>
    </select>
    <br>
    <label for="envs">Environment Variables:</label>
    <textarea id="envs" v-model="runEnvsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
    <br>

    <!-- Start Form -->
    <h3>Start Configuration</h3>

    <label for="appPort">App Port:</label>
    <input type="text" id="appPort" v-model="formData.start.app_port" />
    <br>
    <label for="envs">Environment Variables:</label>
    <textarea id="envs" v-model="startEnvsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>

    <div v-for="(mount, index) in formData.start.mounts" :key="index">
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
    <button @click="submitCombinedForm">Submit Combined Form</button>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
export default {
  setup() {
    const formData = ref({
      run: {
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
      },
      start: {
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
      checkpoint: {
        leave_running: false,
        image_url: '',
        passphrase_file: '',
        preserved_paths: '',
        num_shards: 3,
        cpu_budget: 'medium',
        verbose: 1,
        envs: [],
      },
      stop: true,
    });
    const runEnvsText = ref('');
    const chkEnvsText = ref('');
    const startEnvsText = ref('');
    const capsText = ref('');
    const serviceName = ref('');
    const srcWorker = ref('');
    const destWorker = ref('');

    const addMount = () => {
      formData.value.start.mounts.push({
        type: 'bind',
        source: '',
        target: '',
        readonly: false,
      });
    };
    const removeMount = (index) => {
      formData.value.start.mounts.splice(index, 1);
    };
    const submitCombinedForm = async () => {
      formData.value.run.envs = runEnvsText.value.split('\n').filter(env => env.trim() !== '');
      // Combine form data into a single JSON object
      formData.value.checkpoint.envs = chkEnvsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.start.envs = startEnvsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.start.caps = capsText.value.split('\n').filter(cap => cap.trim() !== '');
      const combinedFormData = {
        ropt: formData.value.run,
        sopt: formData.value.start,
        copt: formData.value.checkpoint,
        stop: formData.value.stop,
      };

      console.log('Combined Form Data:', combinedFormData);
      try {
        const url = `http://localhost:8080/cm_manager/v1.0/migrate/${formData.value.start.container_name}?src=${srcWorker.value}&dest=${destWorker.value}`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(combinedFormData),
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
      formData,
      runEnvsText,
      chkEnvsText,
      startEnvsText,
      capsText,
      serviceName,
      srcWorker,
      destWorker,
      addMount,
      removeMount,
      submitCombinedForm,
    };
  },
};
</script>



<style scoped>
/* Your styles for the OptionsForm component go here */
div {
  background-color: white;
  padding: 10px;
  border-radius: 8px;
  max-height: 80vh;
  /* Set a maximum height, adjust as needed */
  overflow-y: auto;
}

textarea {
  width: 100%;
  height: 80px;
}

.arrow {
  position: relative;
  top: 14px;
  height: 50px;
  width: 45px;
}
</style>
