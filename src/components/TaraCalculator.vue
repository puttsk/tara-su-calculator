<template>
  <div>
    <!-- SU calculator card -->
    <md-card>
      <md-card-header>
        <div class="md-title">TARA SU Calculator</div>
      </md-card-header>
      <md-card-content>
        <table>
          <template v-for="(job, jobId) in jobs">
            <!-- Job label. Show if there is more than 1 job type -->
            <tr v-if="jobs.length > 1" v-bind:key="jobId + '-label'">
              <br />
              <b>Job Type:</b>
              {{
                jobId + 1
              }}
            </tr>
            <!-- End job label.-->
            <!-- Job resource usage -->
            <tr v-bind:key="jobId + '-job'">
              <th>Type</th>
              <th class="num-col">
                <md-field class="col-field">
                  <span class="md-prefix th-field">Time (</span>
                  <md-select v-model="job.timeUnit" md-dense>
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
                    v-model="discountPercent"
                    type="number"
                    style="text-align:right; width:20px;"
                  />
                  <span class="md-suffix th-field">%</span>
                </md-field>
              </th>
            </tr>
            <tr
              v-for="(partition, index) in job.partitions"
              v-bind:key="jobId + '-' + index + '-object'"
            >
              <td>{{ partition.name }}</td>
              <td class="num-col">
                <md-field>
                  <md-input
                    v-model="partition.walltime"
                    type="number"
                    class="number-field"
                  />
                  <span class="md-suffix inline-field">
                    {{ job.timeUnit }}
                  </span>
                </md-field>
              </td>
              <td class="num-col">
                <!--input type="range" v-model.number="t.usage"  /-->
                <md-field>
                  <md-input
                    v-model="partition.usage"
                    type="number"
                    class="number-field"
                  />
                  <span class="md-suffix inline-field" style="width:35px;">
                    {{ partition.unit }}
                  </span>
                </md-field>
              </td>
              <td class="num-col border-left">
                {{
                  (partition.walltime *
                    timeFactor[job.timeUnit] *
                    partition.usage *
                    partition.suFactor)
                    | roundup
                }}
              </td>
              <td class="num-col">
                {{
                  (partition.walltime *
                    timeFactor[job.timeUnit] *
                    partition.usage *
                    partition.suFactor *
                    pricePerSU)
                    | roundup
                }}
              </td>
              <td class="num-col">
                {{
                  ((partition.walltime *
                    timeFactor[job.timeUnit] *
                    partition.usage *
                    partition.suFactor *
                    pricePerSU *
                    (100 - discountPercent)) /
                    100)
                    | roundup
                }}
              </td>
            </tr>
            <!-- End Job resource usage -->

            <!-- Job subtotal. Show if there is more than 1 job type, else add an empty row for better look -->
            <template v-if="jobs.length > 1">
              <tr
                style="line-height:10px;"
                v-bind:key="jobId + '-subtotal-space'"
              >
                <td></td>
                <td></td>
                <td></td>
                <td class="num-col border-left"></td>
                <td>&nbsp;</td>
              </tr>
              <tr v-bind:key="jobId + '-subtotal'" class="row-subtotal">
                <td></td>
                <td></td>
                <td class="num-col"><b>Subtotal:</b></td>
                <td class="num-col border-left subtotal">
                  {{ [job] | sumSU | roundup }}
                </td>
                <td class="num-col subtotal">
                  {{ [job] | sumCost | roundup }}
                </td>
                <td class="num-col subtotal">
                  {{ [job] | sumDiscount | roundup }}
                </td>
              </tr>
            </template>
            <template v-else>
              <tr
                style="line-height:10px;"
                v-bind:key="jobId + '-subtotal-space'"
              >
                <td></td>
                <td></td>
                <td></td>
                <td class="num-col border-left"></td>
                <td>&nbsp;</td>
              </tr>
            </template>
            <!-- End job subtotal.-->
          </template>
          <!-- Summarize all job usage -->
          <tr class="row-sum">
            <td></td>
            <td></td>
            <td class="num-col"><b>Total:</b></td>
            <td class="num-col">{{ jobs | sumSU | roundup }}</td>
            <td class="num-col">{{ jobs | sumCost | roundup }}</td>
            <td class="num-col">{{ jobs | sumDiscount | roundup }}</td>
          </tr>
          <!-- End summarize all job usage -->
        </table>
      </md-card-content>
      <md-card-actions>
        <md-button v-on:click="addJob" class="md-dense md-raised md-primary">
          Add Job Type
        </md-button>
        <md-button v-on:click="reset" class="md-dense md-raised md-primary">
          Reset
        </md-button>
      </md-card-actions>
    </md-card>
    <!-- End SU calculator card -->

    <!-- Pricing card -->
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
          <template v-for="(partition, jobId) in jobTemplate.partitions">
            <tr v-bind:key="jobId + '-type'">
              <td>{{ partition.name }}</td>
              <td class="num-col">{{ partition.unit }}</td>
              <td class="num-col">{{ partition.suFactor }}</td>
            </tr>
          </template>
        </md-table>

        <!-- End pricing card -->
        <br />
        <b>Price per SU: </b> {{ pricePerSU }} THB
      </md-card-content>
    </md-card>
  </div>
</template>

<script>
var jobTemplate = {
  timeUnit: "mins",
  partitions: [
    {
      name: "compute",
      walltime: 1,
      usage: 1,
      suFactor: 1,
      unit: "cores"
    },
    {
      name: "memory",
      walltime: 0,
      usage: 0,
      suFactor: 1.25,
      unit: "cores"
    },
    {
      name: "memory",
      walltime: 0,
      usage: 0,
      suFactor: 0.08,
      unit: "GB"
    },
    {
      name: "memory-preempt",
      walltime: 0,
      usage: 0,
      suFactor: 0.5,
      unit: "cores"
    },
    {
      name: "memory-preempt",
      walltime: 0,
      usage: 0,
      suFactor: 0.032,
      unit: "GB"
    },
    {
      name: "gpu",
      walltime: 0,
      usage: 0,
      suFactor: 130,
      unit: "nodes"
    }
  ]
};

var pricePerSU = 0.0202;
var discountPercent = 50;
var timeFactor = {
  mins: 1,
  hrs: 60,
  days: 60 * 24
};

export default {
  name: "TaraCalculator",
  data: function() {
    return {
      discountPercent: discountPercent,
      pricePerSU: pricePerSU,
      jobs: [JSON.parse(JSON.stringify(jobTemplate))],
      jobTemplate: jobTemplate,
      timeFactor: timeFactor
    };
  },
  props: {},
  filters: {
    roundup: function(value) {
      if (!value) return "0.00";
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    sumSU: function(jobs) {
      var sum = 0;
      for (var i = 0; i < jobs.length; i++) {
        for (var j = 0; j < jobs[i].partitions.length; j++) {
          var partition = jobs[i].partitions[j];
          sum +=
            partition.walltime *
            timeFactor[jobs[i].timeUnit] *
            partition.usage *
            partition.suFactor;
        }
      }
      return sum;
    },
    sumCost: function(jobs) {
      var sum = 0;
      for (var i = 0; i < jobs.length; i++) {
        for (var j = 0; j < jobs[i].partitions.length; j++) {
          var partition = jobs[i].partitions[j];
          sum +=
            partition.walltime *
            timeFactor[jobs[i].timeUnit] *
            partition.usage *
            partition.suFactor *
            pricePerSU;
        }
      }
      return sum;
    },
    sumDiscount: function(jobs) {
      var sum = 0;
      for (var i = 0; i < jobs.length; i++) {
        for (var j = 0; j < jobs[i].partitions.length; j++) {
          var partition = jobs[i].partitions[j];
          sum +=
            partition.walltime *
            timeFactor[jobs[i].timeUnit] *
            partition.usage *
            partition.suFactor *
            pricePerSU;
        }
      }
      return (sum * (100 - discountPercent)) / 100;
    }
  },
  methods: {
    addJob: function() {
      this.jobs.push(JSON.parse(JSON.stringify(jobTemplate)));
    },
    reset: function() {
      this.jobs = [JSON.parse(JSON.stringify(jobTemplate))];
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
td.num-col.subtotal {
  border-top: 1pt solid lightgray;
}
tr.row-subtotal td {
  border-bottom: 1pt solid lightgray;
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
