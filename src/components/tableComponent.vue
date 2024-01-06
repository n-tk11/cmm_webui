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
          <td v-for="(value, colIndex) in row" :key="colIndex">{{ value }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  props: {
    headers: Array, // Array of header names
    rows: Array,    // Array of rows, where each row is an array of values
  },
};
</script>

<style scoped>
/* Add your table styling here */
table {
  width: 500px;
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
</style>