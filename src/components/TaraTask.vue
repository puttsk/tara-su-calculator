<template>
  <div>
    <table style="border-collapse:collapse;">
      <template v-for="(task, taskId) in tasks">
        <tr
          v-if="tasks.length > 1"
          v-bind:key="taskId"
          style="line-height: 2em;"
        >
          <b>Job Type:</b>
          {{
            taskId + 1
          }}
        </tr>
        <tr v-bind:key="taskId">
          <th style="width:100px;">Type</th>
          <th>Time (Min)</th>
          <th>Count</th>
          <th style="width:120px;" class="total-col">SU</th>
          <th style="width:120px;" class="total-col">Cost (THB)</th>
        </tr>
        <tr v-for="(t, index) in task" v-bind:key="index">
          <td>{{ t.nodeType }}</td>
          <td><input v-model="t.nodeTime" /></td>
          <td><input v-model="t.nodeCount" /> {{ t.nodeUnit }}</td>
          <td class="total-col">
            {{ (t.nodeTime * t.nodeCount * t.nodeFactor) | roundup }}
          </td>
          <td class="total-col">
            {{
              (t.nodeTime * t.nodeCount * t.nodeFactor * pricePerSU) | roundup
            }}
          </td>
        </tr>
      </template>
      <tr style="line-height:10px;">
        &nbsp;
      </tr>
      <tr class="row-sum">
        <td></td>
        <td></td>
        <td class="total-col"><b>Total:</b></td>
        <td class="total-col">{{ tasks | sumSU | roundup }}</td>
        <td class="total-col">{{ tasks | sumCost | roundup }}</td>
      </tr>
    </table>

    <button v-on:click="addTask">Add Job Type</button>
    <button v-on:click="reset">Reset</button>
  </div>
</template>

<script>
var taskTemplate = [
  {
    nodeType: "compute",
    nodeTime: 1,
    nodeCount: 1,
    nodeFactor: 1,
    nodeUnit: "cores"
  },
  {
    nodeType: "memory",
    nodeTime: 1,
    nodeCount: 1,
    nodeFactor: 1.25,
    nodeUnit: "cores"
  },
  {
    nodeType: "gpu",
    nodeTime: 1,
    nodeCount: 1,
    nodeFactor: 130,
    nodeUnit: "nodes"
  }
];

var pricePerSU = 0.0202;

export default {
  name: "TaraTask",
  data: function() {
    return {
      pricePerSU: pricePerSU,
      tasks: [JSON.parse(JSON.stringify(taskTemplate))]
    };
  },
  props: {},
  filters: {
    roundup: function(value) {
      if (!value) return "";
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    sumSU: function(tasks) {
      var sum = 0;
      for (var i = 0; i < tasks.length; i++) {
        for (var j = 0; j < tasks[i].length; j++) {
          var t = tasks[i][j];
          sum += t.nodeTime * t.nodeCount * t.nodeFactor;
        }
      }
      return sum;
    },
    sumCost: function(tasks) {
      var sum = 0;
      for (var i = 0; i < tasks.length; i++) {
        for (var j = 0; j < tasks[i].length; j++) {
          var t = tasks[i][j];
          sum += t.nodeTime * t.nodeCount * t.nodeFactor * pricePerSU;
        }
      }
      return sum;
    }
  },
  methods: {
    addTask: function() {
      this.tasks.push(JSON.parse(JSON.stringify(taskTemplate)));
    },
    reset: function() {
      this.tasks = [JSON.parse(JSON.stringify(taskTemplate))];
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.total-col {
  text-align: right;
}
tr.row-sum td {
  padding-top: 0.5em;
  border-top: 1pt solid black;
}
</style>
