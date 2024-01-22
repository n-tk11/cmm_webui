<template>
  <div>
    <h2>Add Worker Form</h2>
    <label for="workerName">Worker Name:</label>
    <input type="text" id="workerName" v-model="worker.worker_id">
    <label for="addr">Worker address and port:</label>
    <input type="text" id="addr" v-model="worker.addr" placeholder="10.0.1.11:7878">
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
    const worker = ref({
      worker_id: '',
      addr: '',
    });
    const root_url = import.meta.env.VITE_API_URL;
    const submitForm = async () => {
      try {
        const url = `${root_url}/worker`;
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(worker.value),
        });
        if (response.ok) {
          console.log('Form submitted successfully');
          const msg = `Worker ${worker.value.worker_id} with addr ${worker.value.addr} is added`;
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



    return {
      worker,
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
