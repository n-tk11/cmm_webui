<template>
  <div>
    <h2>Migrate Configuration Form</h2>
    <div style="padding-top: 0px; margin-left: 0px; width:40%; display: table;">
      <div style="margin-left: 0px; display: table-row;">
        <div style="display: table-cell; text-align: center;">
          <label for="sourceName">Source Worker:</label>
          <select id="sourceName" v-model="srcWorker">
            <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
          </select>
        </div>
        <div style="font-size: 35px;">&rarr;</div>
        <div style="display: table-cell; text-align: center;">
          <label for="destName">Destination Worker:</label>
          <select id="destName" v-model="destWorker">
            <option v-for="worker in workers" :key="worker" :value="worker">{{ worker }}</option>
          </select>
        </div>
      </div>
    </div>
    <div style="padding: 0%; margin-left: 18px;">
      <label for="serviceName">Service:</label>
      <select id="serviceName" v-model="formData.start.container_name" @change="fetchConfig">
        source
        <option v-for="service in services" :key="service" :value="service">{{ service }}</option>
      </select>
    </div>

    <!-- Checkpoint Form -->
    <h3>Checkpoint Configuration(at source)</h3>
    <label for="leaveRunning">Leave Running:</label>
    <input type="checkbox" id="leaveRunning" v-model="formData.checkpoint.leave_running" />
    <br>
    <button @click="toggleVisibility('chk')">
      <span v-if="!isChkVisible">▼</span>
      <span v-else>▲</span>
      Advance Checkpoint Config
    </button>
    <div v-if="isChkVisible">
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
    </div>
    <br>
    <!-- Start Form -->
    <h3>Start Configuration(at source)</h3>

    <label for="appPort">App Port:</label>
    <textarea id="appPort" v-model="app_ports" placeholder="8080:8080&#10;7777:7777"></textarea>
    <br>

    <div v-for="(mount, index) in formData.start.mounts" :key="index">
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
    <button @click="toggleVisibility('str')">
      <span v-if="!isStrVisible">▼</span>
      <span v-else>▲</span>
      Advance Start Config
    </button>
    <div v-if="isStrVisible">
      <label for="envs">Environment Variables:</label>
      <textarea id="envs" v-model="startEnvsText" placeholder="ENV1=value1&#10;ENV2=value2"></textarea>
      <br>
      <label for="caps">Capabilities:</label>
      <textarea id="caps" v-model="capsText" placeholder="CAP1&#10;CAP2"></textarea>
    </div>
    <!-- Run Form -->
    <h3>Run Configuration(at destination)</h3>
    <label for="appArgs">App Arguments:</label>
    <input type="text" id="appArgs" style="width: 300px;" v-model="formData.run.app_args" />
    <br>
    <button @click="toggleVisibility('run')" style="margin-bottom: 5px;">
      <span v-if="!isRunVisible">▼</span>
      <span v-else>▲</span>
      Advance Run Config
    </button>
    <div v-if="isRunVisible">
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
    </div>
    <br><br>
    <label for="stop">Stop Service at Source:</label>
    <input type="checkbox" id="stop" v-model="formData.stop" />
    <br>
    <LoadingSpinner :isLoading="is_Loading" />
    <button @click="submitCombinedForm">Submit Migration Form</button>
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
      stop: false,
    });
    const runEnvsText = ref('');
    const chkEnvsText = ref('');
    const startEnvsText = ref('');
    const capsText = ref('');
    const app_ports = ref('');
    const serviceName = ref('');
    const srcWorker = ref('');
    const destWorker = ref('');
    const root_url = import.meta.env.VITE_API_URL;
    const addMount = () => {
      formData.value.start.mounts.push({
        Type: 'bind',
        Source: '',
        Target: '',
        ReadOnly: false,
      });
    };
    const removeMount = (index) => {
      formData.value.start.mounts.splice(index, 1);
    };
    const submitCombinedForm = async () => {
      is_Loading.value = true;
      formData.value.run.envs = runEnvsText.value.split('\n').filter(env => env.trim() !== '');
      // Combine form data into a single JSON object
      formData.value.checkpoint.envs = chkEnvsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.start.envs = startEnvsText.value.split('\n').filter(env => env.trim() !== '');
      formData.value.start.caps = capsText.value.split('\n').filter(cap => cap.trim() !== '');
      formData.value.start.app_ports = app_ports.value.split('\n').filter(port => port.trim() !== '');
      const combinedFormData = {
        ropt: formData.value.run,
        sopt: formData.value.start,
        copt: formData.value.checkpoint,
        stop: formData.value.stop,
      };

      console.log('Combined Form Data:', combinedFormData);
      try {
        const url = `${root_url}/migrate/${formData.value.start.container_name}?src=${srcWorker.value}&dest=${destWorker.value}`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(combinedFormData),
        });
        is_Loading.value = false;
        if (response.ok) {
          console.log('Form submitted successfully');
          const resp = await response.json();
          const dur = resp.duration;
          const msg = 'Service' + '(' + formData.value.start.container_name + ')' + ' migrated successfully from ' + srcWorker.value + ' to ' + destWorker.value + '!' + '( Duration: ' + dur + 's)';
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

    const fetchConfig = async () => {
      try {
        const url = `${root_url}/service/${formData.value.start.container_name}/config`;
        const response = await fetch(url);
        if (response.ok) {
          const data = await response.json();
          formData.value.run = data.run_opt;
          formData.value.start = data.start_opt;
          formData.value.checkpoint = data.chk_opt;
          app_ports.value = data.start_opt.app_ports.join('\n');
          runEnvsText.value = data.run_opt.envs.join('\n');
          chkEnvsText.value = data.chk_opt.envs.join('\n');
          startEnvsText.value = data.start_opt.envs.join('\n');
          capsText.value = data.start_opt.caps.join('\n');
        } else {
          throw new Error('Request services failed!');
        }
      } catch (error) {
        console.log(error);
      }
    };
    const isChkVisible = ref(false);
    const isRunVisible = ref(false);
    const isStrVisible = ref(false);
    const toggleVisibility = (sec) => {
      switch (sec) {
        case "chk":
          isChkVisible.value = !isChkVisible.value;
          break;
        case "run":
          isRunVisible.value = !isRunVisible.value;
          break;
        case "str":
          isStrVisible.value = !isStrVisible.value;
          break;
      }
    };
    onMounted(fetchData);

    return {
      is_Loading,
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
      fetchConfig,
      toggleVisibility,
      isChkVisible,
      isRunVisible,
      isStrVisible,
      app_ports,
    };
  },
});
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
