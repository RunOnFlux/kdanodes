<template>
  <div>
    <el-tabs v-model="activeName">
      <el-tab-pane
        label="All Nodes"
        name="all"
      >
        <el-table
          :data="allNodes"
          empty-text="No nodes found"
          style="width: 100%"
        >
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
            </template>
          </el-table-column>
          <el-table-column
            label="IP address"
            prop="ip"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Accuracy"
            prop="roundTime"
            sortable
          >
            <template slot-scope="props">
              {{ new Date(props.row.roundTime).toLocaleString('en-GB', timeoptions) }}
            </template>
          </el-table-column>
          <el-table-column
            label="Min. Height"
            prop="height"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Version"
            prop="hash"
            sortable
          >
            <template slot-scope="props">
              <p v-if="props.row.hash === 'localSpecificationsVersion2'">
                2.2
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane
        label="Daily Eligible Nodes"
        name="eligibleA"
      >
        <el-table
          :data="eligibleNodesOne"
          empty-text="No nodes found"
          style="width: 100%"
        >
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
            </template>
          </el-table-column>
          <el-table-column
            label="IP address"
            prop="ip"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Accuracy"
            prop="roundTime"
            sortable
          >
            <template slot-scope="props">
              {{ new Date(props.row.roundTime).toLocaleString('en-GB', timeoptions) }}
            </template>
          </el-table-column>
          <el-table-column
            label="Min. Height"
            prop="height"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Version"
            prop="hash"
            sortable
          >
            <template slot-scope="props">
              <p v-if="props.row.hash === 'localSpecificationsVersion2'">
                2.2
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane
        label="Weekly Eligible Nodes"
        name="eligibleB"
      >
        <el-table
          :data="eligibleNodesSeven"
          empty-text="No nodes found"
          style="width: 100%"
        >
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
            </template>
          </el-table-column>
          <el-table-column
            label="IP address"
            prop="ip"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Accuracy"
            prop="roundTime"
            sortable
          >
            <template slot-scope="props">
              {{ new Date(props.row.roundTime).toLocaleString('en-GB', timeoptions) }}
            </template>
          </el-table-column>
          <el-table-column
            label="Min. Height"
            prop="height"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Version"
            prop="hash"
            sortable
          >
            <template slot-scope="props">
              <p v-if="props.row.hash === 'localSpecificationsVersion2'">
                2.2
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane
        label="Monthly Eligible Nodes"
        name="eligibleC"
      >
        <el-table
          :data="eligibleNodesThirty"
          empty-text="No nodes found"
          style="width: 100%"
        >
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
            </template>
          </el-table-column>
          <el-table-column
            label="IP address"
            prop="ip"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Accuracy"
            prop="roundTime"
            sortable
          >
            <template slot-scope="props">
              {{ new Date(props.row.roundTime).toLocaleString('en-GB', timeoptions) }}
            </template>
          </el-table-column>
          <el-table-column
            label="Min. Height"
            prop="height"
            sortable
          >
          </el-table-column>
          <el-table-column
            label="Version"
            prop="hash"
            sortable
          >
            <template slot-scope="props">
              <p v-if="props.row.hash === 'localSpecificationsVersion2'">
                2.2
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'NodeTracker',
  data() {
    return {
      timeoptions: {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      },
      activeName: 'all',
      baseURL: 'http://localhost:8123/kadena/',
      allNodes: [],
      eligibleNodesOne: [],
      eligibleNodesSeven: [],
      eligibleNodesThirty: [],
    };
  },
  computed: {
  },
  watch: {
    activeName(val, oldVal) {
      console.log(val, oldVal);
      this.switcher(val);
    },
  },
  mounted() {
    this.switcher(this.activeName);
  },
  methods: {
    switcher(value) {
      switch (value) {
        case 'all':
          this.getAll();
          break;
        case 'eligibleA':
          this.getEligible(1);
          break;
        case 'eligibleB':
          this.getEligible(7);
          break;
        case 'eligibleC':
          this.getEligible(30);
          break;
        default:
          console.log('Explorer Section: Unrecognized method');
      }
    },
    async getAll() {
      const res = await axios.get(`${this.baseURL}allnodes`);
      this.allNodes = res.data.data;
    },
    async getEligible(days) {
      const res = await axios.get(`${this.baseURL}eligible/${days}`);
      if (days === 1) {
        this.eligibleOne = res.data.data;
      }
      if (days === 7) {
        this.eligibleSeven = res.data.data;
      }
      if (days === 30) {
        this.eligibleThirty = res.data.data;
      }
    },
  },
};
</script>
