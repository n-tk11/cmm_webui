<template>
  <div class="home">
    <Nav @open-form="openForm"></Nav>
    <div class="div-3">
      <div class="text-wrapper-15">Manager: {{ managerAddr }}</div>
      <div class="text-wrapper-16" :style="{ color: getStatusColor(managerStatus) }">Status: {{ managerStatus }}</div>
      <div class="worker-container">
        <div class="text-wrapper-17">Workers</div>
        <div class="table-container">
          <WTable :headers="tableHeaders" :rows="tableRows">
          </WTable>
          <div class="button-cover">
            <div class="button-container">
              <button @click="openForm('addWorker')">Add a worker</button>
            </div>
            <div class="button-container">
              <button @click="openForm('deleteWorker')">Delete a worker</button>
            </div>
          </div>
        </div>
      </div>
      <!-- <TableWrapper /> -->

      <div class="service-container">
        <div class="text-wrapper-18">Services</div>
        <div class="table-container" style="top: 10px;">
          <Table :headers="servTableHeaders" :rows="servTableRows"></Table>
          <div class="button-cover">
            <div class="button-container">
              <button @click="openForm('addService')">Add a service</button>
            </div>
            <div class="button-container">
              <button @click="openForm('deleteService')">Delete a service</button>
            </div>
          </div>
        </div>
      </div>
      <!-- <AddButtonWrapper /> -->
      <!-- <DivWrapper /> -->
      <!-- <Div /> -->
    </div>
    <FormContainer v-if="showFormOverlay" :formType="currentFormType" @close-form="closeForm" @form-submitted="fetchData">
    </FormContainer>
  </div>
  <RouterView />
</template>

<script setup>
// import checkpointOverlay from './components/checkpointOverlay.vue';
import Nav from './components/navComponent.vue'
import { RouterLink, RouterView } from 'vue-router';
import AddButton from './components/addButtonComponent.vue';
import Table from './components/tableComponent.vue';
import WTable from './components/WorkertableComponent.vue';
import FormContainer from './components/formContainer.vue';
import LoadingSpinner from './components/LoadingSpinner.vue';
import { ref, onMounted } from 'vue';
const tableHeaders = ['Name', 'Address', 'Services', 'Status']
const servTableHeaders = ['Name', 'Checkpoints', 'Image']
const tableRows = ref([]);
const servTableRows = ref([]);
const managerStatus = ref('');
const root_url = import.meta.env.VITE_API_URL;
const managerAddr = import.meta.env.VITE_MANAGER;

const checkIsUp = async () => {
  try {
    const url = `${root_url}/up`;
    const response = await fetch(url);
    if (response.status === 200) {
      managerStatus.value = 'up';
      console.log('Manager is up');
    } else {
      managerStatus.value = 'down';
      console.log('Manager is down');
    }
  } catch (error) {
    console.error('Error fetching data:', error);
    managerStatus.value = 'down';
  }
};

const fetchData = async () => {
  try {
    const url = `${root_url}/worker`;
    const response = await fetch(url);
    const data = await response.json();
    if (data === null || data.length === 0) {
      tableRows.value = [];
      return;
    }
    tableRows.value = data.sort((a, b) => a.id.localeCompare(b.id)).map(item => [item.id, item.addr, item.services, item.status]);
    console.log(data);

    const url2 = `${root_url}/service`;
    const response2 = await fetch(url2);
    const data2 = await response2.json();
    if (data2 === null || data2.length === 0) {
      servTableRows.value = [];
      return;
    }
    servTableRows.value = data2.sort((a, b) => a.name.localeCompare(b.name)).map(item => {
      const checkpoint = item.chk_files.map(file => {
        const [_, worker, time] = file.split('_');
        const localTime = new Date(time).toLocaleString('en-Gb');
        return `${localTime}@${worker}`;
      }).reverse();
      return [item.name, checkpoint, item.image];
    });
    console.log(data2);

  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

onMounted(() => {
  checkIsUp();
  fetchData();
});

const showFormOverlay = ref(false);
const currentFormType = ref('');

const openForm = (route) => {
  showFormOverlay.value = true;
  currentFormType.value = route;
}
const closeForm = () => {
  showFormOverlay.value = false;
  currentFormType.value = '';
}
const getStatusColor = (status) => {
  if (status === 'up') {
    return 'green';
  } else {
    return 'red';
  }
}
</script>

<style scoped>
.home {
  background-color: #ffffff;
  display: flex;
  flex-direction: row;
  justify-content: center;
  width: 100%;
  overflow: hidden;
}

.home .div-3 {
  background-color: #ffffff;
  height: 1024px;
  position: relative;
  width: 1440px;
  left: 100px;
  overflow-y: auto;
}

.home .text-wrapper-15 {
  color: #000000;
  font-family: "Inter-Regular", Helvetica;
  font-size: 30px;
  font-weight: 400;
  letter-spacing: 0;
  line-height: normal;
  position: absolute;
  top: 69px;
  white-space: nowrap;
}

.home .text-wrapper-16 {
  color: #000000;
  font-family: "Inter-Regular", Helvetica;
  font-size: 30px;
  font-weight: 400;
  left: 500px;
  letter-spacing: 0;
  line-height: normal;
  position: absolute;
  top: 69px;
  white-space: nowrap;
}

.home .text-wrapper-17 {
  color: #000000;
  font-family: "Inter-Regular", Helvetica;
  font-size: 30px;
  font-weight: 400;
  letter-spacing: 0;
  line-height: normal;
  position: relative;
  top: 187px;
  white-space: nowrap;
}

.home .text-wrapper-18 {
  color: #000000;
  font-family: "Inter-Regular", Helvetica;
  font-size: 30px;
  font-weight: 400;

  letter-spacing: 0;
  line-height: 39px;
  position: relative;
  white-space: nowrap;
}

.worker-container {
  position: relative;
}

.service-container {
  top: 250px;
  position: relative;
}

.table-container {
  position: relative;
  margin-left: 30px;
  top: 200px;
  /* Adjust the indentation as needed */
}

.button-cover {
  display: flex;
  flex-direction: row;
  gap: 10px;
}

.button-container {
  position: relative;
  top: 10px;
}
</style>
