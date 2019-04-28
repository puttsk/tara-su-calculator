<template>
  <div>
    <md-card>
      <md-card-header>
        <div class="md-title">TARA SU Calculator</div>
      </md-card-header>
      <md-card-content>
        <table>
          <template v-for="(task, taskId) in tasks">
            <tr v-if="tasks.length > 1" v-bind:key="taskId + '-label'">
              <br />
              <b>Job Type:</b>
              {{
                taskId + 1
              }}
            </tr>
            <tr v-bind:key="taskId + '-job'">
              <th>Type</th>
              <th class="num-col">
                <md-field class="col-field">
                  <span class="md-prefix th-field">Time (</span>
                  <md-select v-model="task.timeUnit" md-dense>
                    <md-option
                      v-for="(val, key) in timeFactor"
                      :value="key"
                      :key="key"
                    >
                      {{ key }}
                    </md-option>
                  </md-select>
                  <span class="md-suffix th-field">)</span>
                </md-field>
              </th>
              <th class="num-col">Usage</th>
              <th class="num-col long">SU</th>
              <th class="num-col long">Cost (THB)</th>
              <th class="num-col long">
                Discount
                <md-field>
                  <md-input
                    v-model="discount"
                    type="number"
                    style="text-align:right; width:20px;"
                  />
                  <span
                    class="md-suffix th-field"
                  >
                    %
                  </span>
                </md-field>
              </th>
            </tr>
            <tr
              v-for="(t, index) in task.partitionClass"
              v-bind:key="taskId + '-' + index + '-object'"
            >
              <td>{{ t.nodeType }}</td>
              <td class="num-col">
                <md-field>
                  <md-input
                    v-model="t.nodeTime"
                    type="number"
                    class="number-field"
                  />
                  <span
                    class="md-suffix inline-field"
                  >
                    {{ task.timeUnit }}
                  </span>
                </md-field>
              </td>
              <td class="num-col">
                <!--input type="range" v-model.number="t.nodeCount"  /-->
                <md-field>
                  <md-input
                    v-model="t.nodeCount"
                    type="number"
                    class="number-field"
                  />
                  <span
                    class="md-suffix inline-field"
                    style="width:35px;"
                  >
                    {{ t.nodeUnit }}
                  </span>
                </md-field>
              </td>
              <td class="num-col border-left">
                {{
                  (t.nodeTime *
                    timeFactor[task.timeUnit] *
                    t.nodeCount *
                    t.nodeFactor)
                    | roundup
                }}
              </td>
              <td class="num-col">
                {{
                  (t.nodeTime *
                    timeFactor[task.timeUnit] *
                    t.nodeCount *
                    t.nodeFactor *
                    pricePerSU)
                    | roundup
                }}
              </td>
              <td class="num-col">
                {{
                  ((t.nodeTime *
                    timeFactor[task.timeUnit] *
                    t.nodeCount *
                    t.nodeFactor *
                    pricePerSU *
                    (100 - discount)) /
                    100)
                    | roundup
                }}
              </td>
            </tr>
          </template>
          <tr style="line-height:10px;">
            <td></td>
            <td></td>
            <td></td>
            <td class="num-col border-left"></td>
            <td>&nbsp;</td>
          </tr>
          <tr class="row-sum">
            <td></td>
            <td></td>
            <td class="num-col"><b>Total:</b></td>
            <td class="num-col">{{ tasks | sumSU | roundup }}</td>
            <td class="num-col">{{ tasks | sumCost | roundup }}</td>
            <td class="num-col">{{ tasks | sumDiscount | roundup }}</td>
          </tr>
        </table>
      </md-card-content>
      <md-card-actions>
        <md-button v-on:click="addTask" class="md-dense md-raised md-primary">
          Add Job Type
        </md-button>
        <md-button v-on:click="reset" class="md-dense md-raised md-primary">
          Reset
        </md-button>
      </md-card-actions>
    </md-card>

    <md-card>
      <md-card-header>
        <div class="md-title">Pricing</div>
      </md-card-header>
      <md-card-content>
        <md-table>
          <tr>
            <th style="min-width:140px;">Type</th>
            <th class="num-col">Unit</th>
            <th class="num-col long">SU/Unit-Min</th>
          </tr>
          <template v-for="(t, taskId) in taskTemplate.partitionClass">
            <tr v-bind:key="taskId + '-type'">
              <td>{{ t.nodeType }}</td>
              <td class="num-col">{{ t.nodeUnit }}</td>
              <td class="num-col">{{ t.nodeFactor }}</td>
            </tr>
          </template>
        </md-table>

        <br />
        <b>Price per SU: </b> {{ pricePerSU }} THB
      </md-card-content>
    </md-card>
  </div>
</template>

<script>
var taskTemplate = {
  timeUnit: "mins",
  partitionClass: [
    {
      nodeType: "compute",
      nodeTime: 1,
      nodeCount: 1,
      nodeFactor: 1,
      nodeUnit: "cores"
    },
    {
      nodeType: "memory",
      nodeTime: 0,
      nodeCount: 0,
      nodeFactor: 1.25,
      nodeUnit: "cores"
    },
    {
      nodeType: "memory",
      nodeTime: 0,
      nodeCount: 0,
      nodeFactor: 0.08,
      nodeUnit: "GB"
    },
    {
      nodeType: "memory-preempt",
      nodeTime: 0,
      nodeCount: 0,
      nodeFactor: 0.5,
      nodeUnit: "cores"
    },
    {
      nodeType: "memory-preempt",
      nodeTime: 0,
      nodeCount: 0,
      nodeFactor: 0.032,
      nodeUnit: "GB"
    },
    {
      nodeType: "gpu",
      nodeTime: 0,
      nodeCount: 0,
      nodeFactor: 130,
      nodeUnit: "nodes"
    }
  ]
};

var pricePerSU = 0.0202;
var discount = 50;
var timeFactor = {
  mins: 1,
  hrs: 60,
  days: 60 * 24
};

export default {
  name: "TaraTask",
  data: function() {
    return {
      discount: discount,
      pricePerSU: pricePerSU,
      tasks: [JSON.parse(JSON.stringify(taskTemplate))],
      taskTemplate: taskTemplate,
      timeFactor: timeFactor
    };
  },
  props: {},
  filters: {
    roundup: function(value) {
      if (!value) return "0.00";
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    sumSU: function(tasks) {
      var sum = 0;
      for (var i = 0; i < tasks.length; i++) {
        for (var j = 0; j < tasks[i].partitionClass.length; j++) {
          var t = tasks[i].partitionClass[j];
          sum +=
            t.nodeTime *
            timeFactor[tasks[i].timeUnit] *
            t.nodeCount *
            t.nodeFactor;
        }
      }
      return sum;
    },
    sumCost: function(tasks) {
      var sum = 0;
      for (var i = 0; i < tasks.length; i++) {
        for (var j = 0; j < tasks[i].partitionClass.length; j++) {
          var t = tasks[i].partitionClass[j];
          sum +=
            t.nodeTime *
            timeFactor[tasks[i].timeUnit] *
            t.nodeCount *
            t.nodeFactor *
            pricePerSU;
        }
      }
      return sum;
    },
    sumDiscount: function(tasks) {
      var sum = 0;
      for (var i = 0; i < tasks.length; i++) {
        for (var j = 0; j < tasks[i].partitionClass.length; j++) {
          var t = tasks[i].partitionClass[j];
          sum +=
            t.nodeTime *
            timeFactor[tasks[i].timeUnit] *
            t.nodeCount *
            t.nodeFactor *
            pricePerSU;
        }
      }
      return (sum * (100 - discount)) / 100;
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
table {
  border-collapse: separate;
  border-spacing: 0px;
}
table td {
  text-align: left;
}
table th {
  text-align: left;
  border-bottom: 1pt solid lightgray;
  min-width: 80px;
}
th.num-col,
td.num-col {
  text-align: right;
  padding: 0 20px 0 5px;
}
th.num-col.long,
td.num-col.long {
  min-width: 100px;
}
th.num-col.border-left,
td.num-col.border-left {
  border-left: 1pt solid lightgray;
}
tr.row-sum td {
  padding-top: 0.5em;
  border-top: 1pt solid black;
}
th .md-field,
td .md-field {
  min-height: 0px;
  margin: 0px;
  margin-top: -2px;
  padding-top: 0px;
}
.md-card {
  width: auto;
  margin: 4px;
  padding: 10px;
  display: inline-block;
  vertical-align: top;
}
.md-field.col-field {
  width: 120px;
  text-align: right;
  margin-bottom: -6px;
}
.md-field .md-prefix.th-field, 
.md-field .md-suffix.th-field {
  font-size: 14px;
  color: black;
}
.md-field .md-suffix.inline-field {
  text-align: right;
  font-size: small;
}
.md-field .md-input.number-field {
  text-align: right;
  width: 100px;
}
</style>
