<template>
  <div>
    <table>
      <thead>
        <tr>
          <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, rowIndex) in rows" :key="rowIndex">
          <td v-for="(value, colIndex) in row" :key="colIndex">
            <!-- Check if the content is an array -->
            <template v-if="Array.isArray(value)">
              <!-- Display array as a list with collapse/expand button -->
              <div>
                <ul>
                  <li v-for="(listItem, listIndex) in value.slice(0, 3)" :key="listIndex">
                    Name:{{ listItem.name }}
                    Status:<div :style="{ color: getStatusColor(listItem.status) }">{{ listItem.status }}</div>

                  </li>
                </ul>
                <button @click=" toggleList(rowIndex, colIndex)" v-if="value.length > 3">
                  {{ isListVisible(rowIndex, colIndex) ? 'Less' : 'More' }}
                </button>
                <ul v-if="isListVisible(rowIndex, colIndex)" style="margin: 0px;">
                  <li v-for="(listItem, listIndex) in value.slice(3)" :key="listIndex">
                    Name:{{ listItem.name }}
                    Status:<div :style="{ color: getStatusColor(listItem.status) }">{{ listItem.status }}</div>
                  </li>
                </ul>
              </div>
            </template>
            <!-- If content is not an array, display it normally -->
            <template v-else>
              <div :style="{ color: getStatusColor(value) }">{{ value }}</div>
            </template>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import Circle from './statusCircle.vue';
export default {
  components: {
    Circle,
  },
  props: {
    headers: Array, // Array of header names
    rows: Array,    // Array of rows, where each row is an array of values
  },
  methods: {
    toggleList(rowIndex, colIndex) {
      const key = `${rowIndex}-${colIndex}`;
      this.$data.collapsedLists[key] = !this.isListVisible(rowIndex, colIndex);
      this.$forceUpdate();
    },
    isListVisible(rowIndex, colIndex) {
      const key = `${rowIndex}-${colIndex}`;
      return this.collapsedLists[key];
    },
    getStatusColor(status) {
      switch (status) {
        case 'stopped':
          return 'red'; // Change this to your desired color for stop status
        case 'running':
          return 'green'; // Change this to your desired color for run status
        // Add more cases based on your actual status values
        case 'standby':
          return 'orange';
        case 'exited':
          return 'red';
        case 'up':
          return 'green';
        case 'down':
          return 'red';
        case 'checkpointed':
          return 'orange';
        default:
          return 'black'; // Default color for other statuses
      }
    },
  },
  data() {
    const initialCollapsedLists = {};
    this.rows.forEach((_, rowIndex) => {
      this.headers.forEach((_, colIndex) => {
        initialCollapsedLists[`${rowIndex}-${colIndex}`] = true;
      });
    });

    return {
      collapsedLists: initialCollapsedLists,
    };
  },

};
</script>

<style scoped>
/* Add your table styling here */
table {
  width: 750px;
  border-collapse: collapse;
  margin-top: 20px;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
  table-layout: auto;

}

th {
  background-color: #f2f2f2;
}

.stop-status .exit-status {
  color: red;
}

.standby-status {
  color: orange;
}

.run-status {
  color: green;
}

li {
  display: flex;
  align-items: center;
  margin-bottom: 5px;
  /* Add margin for spacing between list items */
}
</style>