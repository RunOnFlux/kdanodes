<template>
  <div>
    <el-tabs v-model="activeName">
      <el-tab-pane
        label="Active Nodes"
        name="all"
      >
        <el-table
          :data="allNodes.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15')))"
          empty-text="No nodes found"
          style="width: 100%"
          @expand-change="loadEligibility"
        >
          <div slot="empty">
            <p v-if="loadingAll">
              Loading Nodes <i class="el-icon-loading"></i>
            </p>
            <p v-else-if="filterNodes">
              No Nodes Match Filter
            </p>
            <p v-else>
              No Nodes
            </p>
          </div>
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
              <p :key="updateKey">Last Day Uptime: {{ !loadingHistory[props.row.ip] ? lastDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Weekly Uptime: {{ !loadingHistory[props.row.ip] ? sevenDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Monthly Uptime: {{ !loadingHistory[props.row.ip] ? thirtyDayUptime(props.row.ip) : 'loading' }}%</p>
              <el-switch
                v-model="showHistory[props.row.ip]"
                active-text="Show History"
              >
              </el-switch>
              <el-table
                v-if="showHistory[props.row.ip]"
                :data="limitedHistory[props.row.ip].reverse()"
                :key="updateKey"
                :show-header="false"
                style="width: 100%"
              >
                <div slot="empty">
                  <p v-if="loadingHistory[props.row.ip]">
                    Loading History <i class="el-icon-loading"></i>
                  </p>
                  <p v-else>
                    No History Found
                  </p>
                </div>
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <p>Zel ID: {{ props.row.zelid }}</p>
                    <p>Kadena: {{ props.row.account }}</p>
                    <p>Height: {{ props.row.height }}</p>
                    <p>Tier: {{ props.row.tier }}</p>
                    <p>Hash: {{ props.row.hash }}</p>
                  </template>
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
              </el-table>
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
                2.2 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion3'">
                2.3 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion4'">
                2.4
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion5'">
                2.4.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion6'">
                2.5
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion7'">
                2.6
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion8'">
                2.7
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion9'">
                2.8
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion10'">
                2.9
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion11'">
                2.9.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion12'">
                2.9.2
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion13'">
                2.10.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion14'">
                2.11.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion15'">
                2.12.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion16'">
                2.12.1
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template
              slot="header"
              slot-scope="scope"
            >
              <el-row :gutter="10">
                <el-col
                  :xs="4"
                  :sm="4"
                  :md="4"
                  :lg="4"
                  :xl="4"
                >
                  {{ allNodes.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15'))).length }}
                </el-col>
                <el-col
                  :xs="2"
                  :sm="2"
                  :md="2"
                  :lg="2"
                  :xl="2"
                >
                  <el-checkbox v-model="hasAccountAndCorrectVersion"></el-checkbox>
                </el-col>
                <el-col
                  :xs="18"
                  :sm="18"
                  :md="18"
                  :lg="18"
                  :xl="18"
                >
                  <el-input
                    v-if="scope"
                    v-model="filterNodes"
                    size="mini"
                    placeholder="Type to search"
                  />
                </el-col>
              </el-row>
            </template>
            <template slot-scope="props">
              <p v-if="props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid=')">
                OK
              </p>
              <p v-else>
                Missing Account
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
          :data="eligibleNodesOne.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15')))"
          empty-text="No nodes found"
          style="width: 100%"
          @expand-change="loadEligibility"
        >
          <div slot="empty">
            <p v-if="loadingOne">
              Loading Eligible Nodes <i class="el-icon-loading"></i>
            </p>
            <p v-else-if="filterNodes">
              No Eligible Nodes Match Filter
            </p>
            <p v-else>
              No Eligible Nodes
            </p>
          </div>
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
              <p :key="updateKey">Last Day Uptime: {{ !loadingHistory[props.row.ip] ? lastDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Weekly Uptime: {{ !loadingHistory[props.row.ip] ? sevenDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Monthly Uptime: {{ !loadingHistory[props.row.ip] ? thirtyDayUptime(props.row.ip) : 'loading' }}%</p>
              <el-switch
                v-model="showHistory[props.row.ip]"
                active-text="Show History"
              >
              </el-switch>
              <el-table
                v-if="showHistory[props.row.ip]"
                :data="dayHistory(props.row.ip).reverse()"
                :key="updateKey"
                :show-header="false"
                style="width: 100%"
              >
                <div slot="empty">
                  <p v-if="loadingHistory[props.row.ip]">
                    Loading History <i class="el-icon-loading"></i>
                  </p>
                  <p v-else>
                    No History Found
                  </p>
                </div>
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <p>Zel ID: {{ props.row.zelid }}</p>
                    <p>Kadena: {{ props.row.account }}</p>
                    <p>Height: {{ props.row.height }}</p>
                    <p>Tier: {{ props.row.tier }}</p>
                    <p>Hash: {{ props.row.hash }}</p>
                  </template>
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
              </el-table>
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
                2.2 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion3'">
                2.3 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion4'">
                2.4
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion5'">
                2.4.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion6'">
                2.5
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion7'">
                2.6
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion8'">
                2.7
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion9'">
                2.8
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion10'">
                2.9
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion11'">
                2.9.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion12'">
                2.9.2
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion13'">
                2.10.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion14'">
                2.11.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion15'">
                2.12.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion16'">
                2.12.1
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template
              slot="header"
              slot-scope="scope"
            >
              <el-row :gutter="10">
                <el-col
                  :xs="4"
                  :sm="4"
                  :md="4"
                  :lg="4"
                  :xl="4"
                >
                  {{ eligibleNodesOne.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15'))).length }}
                </el-col>
                <el-col
                  :xs="2"
                  :sm="2"
                  :md="2"
                  :lg="2"
                  :xl="2"
                >
                  <el-checkbox v-model="hasAccountAndCorrectVersion"></el-checkbox>
                </el-col>
                <el-col
                  :xs="18"
                  :sm="18"
                  :md="18"
                  :lg="18"
                  :xl="18"
                >
                  <el-input
                    v-if="scope"
                    v-model="filterNodes"
                    size="mini"
                    placeholder="Type to search"
                  />
                </el-col>
              </el-row>
            </template>
            <template slot-scope="props">
              <p v-if="!(props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid='))">
                Missing Account
              </p>
              <p v-if="props.row.hash !== 'localSpecificationsVersion16' && props.row.hash !== 'localSpecificationsVersion15'">
                Invalid Version
              </p>
              <p v-if="props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid=') && (props.row.hash === 'localSpecificationsVersion16' || props.row.hash === 'localSpecificationsVersion15')">
                Payout Eligible
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
          :data="eligibleNodesSeven.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15')))"
          empty-text="No nodes found"
          style="width: 100%"
          @expand-change="loadEligibility"
        >
          <div slot="empty">
            <p v-if="loadingSeven">
              Loading Eligible Nodes <i class="el-icon-loading"></i>
            </p>
            <p v-else-if="filterNodes">
              No Eligible Nodes Match Filter
            </p>
            <p v-else>
              No Eligible Nodes
            </p>
          </div>
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
              <p :key="updateKey">Last Day Uptime: {{ !loadingHistory[props.row.ip] ? lastDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Weekly Uptime: {{ !loadingHistory[props.row.ip] ? sevenDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Monthly Uptime: {{ !loadingHistory[props.row.ip] ? thirtyDayUptime(props.row.ip) : 'loading' }}%</p>
              <el-switch
                v-model="showHistory[props.row.ip]"
                active-text="Show History"
              >
              </el-switch>
              <el-table
                v-if="showHistory[props.row.ip]"
                :data="weekHistory(props.row.ip).reverse()"
                :key="updateKey"
                :show-header="false"
                style="width: 100%"
              >
                <div slot="empty">
                  <p v-if="loadingHistory[props.row.ip]">
                    Loading History <i class="el-icon-loading"></i>
                  </p>
                  <p v-else>
                    No History Found
                  </p>
                </div>
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <p>Zel ID: {{ props.row.zelid }}</p>
                    <p>Kadena: {{ props.row.account }}</p>
                    <p>Height: {{ props.row.height }}</p>
                    <p>Tier: {{ props.row.tier }}</p>
                    <p>Hash: {{ props.row.hash }}</p>
                  </template>
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
              </el-table>
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
                2.2 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion3'">
                2.3 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion4'">
                2.4
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion5'">
                2.4.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion6'">
                2.5
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion7'">
                2.6
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion8'">
                2.7
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion9'">
                2.8
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion10'">
                2.9
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion11'">
                2.9.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion12'">
                2.9.2
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion13'">
                2.10.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion14'">
                2.11.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion15'">
                2.12.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion16'">
                2.12.1
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template
              slot="header"
              slot-scope="scope"
            >
              <el-row :gutter="10">
                <el-col
                  :xs="4"
                  :sm="4"
                  :md="4"
                  :lg="4"
                  :xl="4"
                >
                  {{ eligibleNodesSeven.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15'))).length }}
                </el-col>
                <el-col
                  :xs="2"
                  :sm="2"
                  :md="2"
                  :lg="2"
                  :xl="2"
                >
                  <el-checkbox v-model="hasAccountAndCorrectVersion"></el-checkbox>
                </el-col>
                <el-col
                  :xs="18"
                  :sm="18"
                  :md="18"
                  :lg="18"
                  :xl="18"
                >
                  <el-input
                    v-if="scope"
                    v-model="filterNodes"
                    size="mini"
                    placeholder="Type to search"
                  />
                </el-col>
              </el-row>
            </template>
            <template slot-scope="props">
              <p v-if="!(props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid='))">
                Missing Account
              </p>
              <p v-if="props.row.hash !== 'localSpecificationsVersion16' && props.row.hash !== 'localSpecificationsVersion15'">
                Invalid Version
              </p>
              <p v-if="props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid=') && (props.row.hash === 'localSpecificationsVersion16' || props.row.hash === 'localSpecificationsVersion15')">
                Payout Eligible
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
          :data="eligibleNodesThirty.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15')))"
          empty-text="No nodes found"
          style="width: 100%"
          @expand-change="loadEligibility"
        >
          <div slot="empty">
            <p v-if="loadingThirty">
              Loading Eligible Nodes <i class="el-icon-loading"></i>
            </p>
            <p v-else-if="filterNodes">
              No Eligible Nodes Match Filter
            </p>
            <p v-else>
              No Eligible Nodes
            </p>
          </div>
          <el-table-column type="expand">
            <template slot-scope="props">
              <p>Zel ID: {{ props.row.zelid }}</p>
              <p>Kadena: {{ props.row.account }}</p>
              <p>Tier: {{ props.row.tier }}</p>
              <p>Hash: {{ props.row.hash }}</p>
              <p :key="updateKey">Last Day Uptime: {{ !loadingHistory[props.row.ip] ? lastDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Weekly Uptime: {{ !loadingHistory[props.row.ip] ? sevenDayUptime(props.row.ip) : 'loading' }}%</p>
              <p :key="updateKey">Monthly Uptime: {{ !loadingHistory[props.row.ip] ? thirtyDayUptime(props.row.ip) : 'loading' }}%</p>
              <el-switch
                v-model="showHistory[props.row.ip]"
                active-text="Show History"
              >
              </el-switch>
              <el-table
                v-if="showHistory[props.row.ip]"
                :data="limitedHistory[props.row.ip].reverse()"
                :key="updateKey"
                :show-header="false"
                style="width: 100%"
              >
                <div slot="empty">
                  <p v-if="loadingHistory[props.row.ip]">
                    Loading History <i class="el-icon-loading"></i>
                  </p>
                  <p v-else>
                    No History Found
                  </p>
                </div>
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <p>Zel ID: {{ props.row.zelid }}</p>
                    <p>Kadena: {{ props.row.account }}</p>
                    <p>Height: {{ props.row.height }}</p>
                    <p>Tier: {{ props.row.tier }}</p>
                    <p>Hash: {{ props.row.hash }}</p>
                  </template>
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
              </el-table>
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
                2.2 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion3'">
                2.3 Outdated
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion4'">
                2.4
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion5'">
                2.4.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion6'">
                2.5
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion7'">
                2.6
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion8'">
                2.7
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion9'">
                2.8
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion10'">
                2.9
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion11'">
                2.9.1
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion12'">
                2.9.2
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion13'">
                2.10.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion14'">
                2.11.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion15'">
                2.12.0
              </p>
              <p v-else-if="props.row.hash === 'localSpecificationsVersion16'">
                2.12.1
              </p>
              <p v-else>
                Outdated
              </p>
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template
              slot="header"
              slot-scope="scope"
            >
              <el-row :gutter="10">
                <el-col
                  :xs="4"
                  :sm="4"
                  :md="4"
                  :lg="4"
                  :xl="4"
                >
                  {{ eligibleNodesThirty.filter(data => !filterNodes || data.ip.toLowerCase().includes(filterNodes.toLowerCase()) || (data.zelid ? data.zelid.toLowerCase().includes(filterNodes.toLowerCase()) : false) || (data.account ? data.account.toLowerCase().includes(filterNodes.toLowerCase()) : false)).filter((dataB) => !hasAccountAndCorrectVersion || (dataB.account && dataB.account.includes('kadena:') && dataB.account.includes('?chainid=') && (dataB.hash === 'localSpecificationsVersion16' || dataB.hash === 'localSpecificationsVersion15'))).length }}
                </el-col>
                <el-col
                  :xs="2"
                  :sm="2"
                  :md="2"
                  :lg="2"
                  :xl="2"
                >
                  <el-checkbox v-model="hasAccountAndCorrectVersion"></el-checkbox>
                </el-col>
                <el-col
                  :xs="18"
                  :sm="18"
                  :md="18"
                  :lg="18"
                  :xl="18"
                >
                  <el-input
                    v-if="scope"
                    v-model="filterNodes"
                    size="mini"
                    placeholder="Type to search"
                  />
                </el-col>
              </el-row>
            </template>
            <template slot-scope="props">
              <p v-if="!(props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid='))">
                Missing Account
              </p>
              <p v-if="props.row.hash !== 'localSpecificationsVersion16' && props.row.hash !== 'localSpecificationsVersion15'">
                Invalid Version
              </p>
              <p v-if="props.row.account && props.row.account.includes('kadena:') && props.row.account.includes('?chainid=') && (props.row.hash === 'localSpecificationsVersion16' || props.row.hash === 'localSpecificationsVersion15')">
                Payout Eligible
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
      baseURL: 'https://api.flux.zel.network/kadena/',
      allNodes: [],
      eligibleNodesOne: [],
      eligibleNodesSeven: [],
      eligibleNodesThirty: [],
      filterNodes: "",
      hasAccountAndCorrectVersion: false,
      loadingAll: false,
      loadingOne: false,
      loadingSeven: false,
      loadingThirty: false,
      limitedHistory: {},
      loadingHistory: {},
      updateKey: 1,
      showHistory: {},
    };
  },
  computed: {
    lastDayMinTime() {
      const daysInMiliseconds = 7 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      return minimumTime;
    },
    lastWeekMinTime() {
      const daysInMiliseconds = 7 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      return minimumTime;
    },
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
      this.loadingAll = true;
      const res = await axios.get(`${this.baseURL}allnodes`);
      this.allNodes = res.data.data;
      this.loadingAll = false;
    },
    async getEligible(days) {
      if (days === 1) {
        this.loadingOne = true;
      }
      if (days === 7) {
        this.loadingSeven = true;
      }
      if (days === 30) {
        this.loadingThirty = true;
      }
      const res = await axios.get(`${this.baseURL}eligible/${days}`);
      if (days === 1) {
        this.eligibleNodesOne = res.data.data;
        this.loadingOne = false;
      }
      if (days === 7) {
        this.eligibleNodesSeven = res.data.data;
        this.loadingSeven = false;
      }
      if (days === 30) {
        this.eligibleNodesThirty = res.data.data;
        this.loadingThirty = false;
      }
    },
    async loadEligibility(row, expanded) {
      const days = 30
      console.log(row);
      console.log(expanded);
      if (this.limitedHistory[row.ip]) {
        return;
      }
      this.loadingHistory[row.ip] = true;
      this.updateKey += 1;
      const response = await await axios.get(`${this.baseURL}limitedhistory/ip/${row.ip}/${days}`);
      console.log(response);
      if (response.data.status === 'success') {
        const limHist = response.data.data;
        this.limitedHistory[row.ip] = limHist;
        this.updateKey += 1;
        this.loadingHistory[row.ip] = false;
      }
    },
    dayHistory(ip) {
      const daysInMiliseconds = 1 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      const validChecks = this.limitedHistory[ip].filter((data) => data.roundTime > minimumTime);
      return validChecks;
    },
    weekHistory(ip) {
      const daysInMiliseconds = 1 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      const validChecks = this.limitedHistory[ip].filter((data) => data.roundTime > minimumTime);
      return validChecks;
    },
    lastDayUptime(ip) {
      if (!this.limitedHistory[ip]) {
        return 0;
      }
      const numberOfChecksPerDay = 1 * 48 - 3; // - 5% for inconsistency
      const daysInMiliseconds = 1 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      const baseTime = 1625422726000;
      const baseHeight = 35347955;
      const timeDifference = currentTime - baseTime;
      const blocksPassedInDifference = (timeDifference / 30000) * 20; // 20 chains with blocktime 30 seconds
      const blocksInTimeFrame = (daysInMiliseconds / 30000) * 20;
      const currentBlockEstimation = baseHeight + blocksPassedInDifference;
      const minimumAcceptedBlockHeight = currentBlockEstimation - blocksInTimeFrame - 200000; // allow being off sync for 200000 blocks;
      const validChecks = this.limitedHistory[ip].filter((data) => data.roundTime > minimumTime && typeof data.tier === "string" && data.height > minimumAcceptedBlockHeight)
      const l = validChecks.length + 1;
      let uptime = l / numberOfChecksPerDay;
      if (uptime > 1) {
        uptime = 1;
      }
      uptime = (uptime * 100).toFixed(2);
      return uptime;
    },
    sevenDayUptime(ip) {
      if (!this.limitedHistory[ip]) {
        return 0;
      }
      const numberOfChecksPerDay = 7 * 48 - 17; // - 5% for inconsistency
      const daysInMiliseconds = 7 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      const baseTime = 1625422726000;
      const baseHeight = 35347955;
      const timeDifference = currentTime - baseTime;
      const blocksPassedInDifference = (timeDifference / 30000) * 20; // 20 chains with blocktime 30 seconds
      const blocksInTimeFrame = (daysInMiliseconds / 30000) * 20;
      const currentBlockEstimation = baseHeight + blocksPassedInDifference;
      const minimumAcceptedBlockHeight = currentBlockEstimation - blocksInTimeFrame - 200000; // allow being off sync for 200000 blocks;
      const validChecks = this.limitedHistory[ip].filter((data) => data.roundTime > minimumTime && typeof data.tier === "string" && data.height > minimumAcceptedBlockHeight)
      const l = validChecks.length;
      let uptime = l / numberOfChecksPerDay;
      if (uptime > 1) {
        uptime = 1;
      }
      uptime = (uptime * 100).toFixed(2);
      return uptime;
    },
    thirtyDayUptime(ip) {
      if (!this.limitedHistory[ip]) {
        return 0;
      }
      const numberOfChecksPerDay = 30 * 48 - 70; // - 5% for inconsistency
      const daysInMiliseconds = 30 * 24 * 60 * 60 * 1000;
      const currentTime = new Date().getTime();
      const minimumTime = currentTime - daysInMiliseconds
      const baseTime = 1625422726000;
      const baseHeight = 35347955;
      const timeDifference = currentTime - baseTime;
      const blocksPassedInDifference = (timeDifference / 30000) * 20; // 20 chains with blocktime 30 seconds
      const blocksInTimeFrame = (daysInMiliseconds / 30000) * 20;
      const currentBlockEstimation = baseHeight + blocksPassedInDifference;
      const minimumAcceptedBlockHeight = currentBlockEstimation - blocksInTimeFrame - 200000; // allow being off sync for 200000 blocks;
      const validChecks = this.limitedHistory[ip].filter((data) => data.roundTime > minimumTime && typeof data.tier === "string" && data.height > minimumAcceptedBlockHeight)
      const l = validChecks.length;
      let uptime = l / numberOfChecksPerDay;
      if (uptime > 1) {
        uptime = 1;
      }
      uptime = (uptime * 100).toFixed(2);
      return uptime;
    },
  },
};
</script>
