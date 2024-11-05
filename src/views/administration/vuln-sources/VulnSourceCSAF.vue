<template>
  <b-card no-body :header="header">
    <b-card-body>
      <!--TODO: Include logo file-->
      <img
        alt="CSAF logo"
        src="https://oasis-open.github.io/csaf-documentation/assets/images/screen-shot-2022-11-28-at-10.16.00-pm-279x97.png"
        width="65"
      />

      <hr />
      <c-switch
        color="primary"
        id="vulnsourceEnabled"
        label
        v-bind="labelIcon"
        v-model="vulnsourceEnabled"
        :disabled="enabledEcosystems.length === 0"
      />
      {{ $t('admin.vulnsource_csaf_advisories_enable') }}
      <hr />
      <div>
        <b-card no-body>
          <b-tabs pills card>
            <b-tab title="CSAF Sources" active>
              <!--<b-card no-body :header="header">-->
              <b-card-body>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    v-b-modal.vulnSourceCSAFAddModal
                  >
                    <span class="fa fa-plus"></span>
                    {{ $t('admin.add_source') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    v-b-modal.vulnSourceCSAFUpload
                  >
                    <span class="fa fa-upload"></span>
                    {{ $t('admin.upload_file') }}
                  </b-button>
                </div>
                <h3>{{ $t('admin.csaf_sources') }}:</h3>
                <bootstrap-table
                  ref="table_sources"
                  :columns="srcCols"
                  :data="srcData"
                  :options="srcOpts"
                >
                </bootstrap-table>
                <h3>{{ $t('admin.suggested_discovery_sources') }}:</h3>
                <bootstrap-table
                  ref="table_suggested"
                  :columns="columns"
                  :data="data"
                  :options="options"
                >
                </bootstrap-table>
                <hr>

              </b-card-body>
              <!--<repository-create-repository-modal
                  :type="type"
                  v-on:refreshTable="refreshTable"
                />-->
              <!--</b-card>-->
            </b-tab>
            <b-tab title="CSAF Documents">
              <b-card-body>
                <h2>{{ $t('admin.csaf_documents') }}:</h2>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="openCompare"
                >
                  <span class="fa fa-file"></span>
                  {{ $t('admin.compare_selected') }}
                </b-button>
                <bootstrap-table
                  ref="table_documents"
                  :columns="columns"
                  :data="data"
                  :options="options"
                  data-click-to-select="true"
                  @check-change="onCheckChange"
                >
                </bootstrap-table>
              </b-card-body>
            </b-tab>
          </b-tabs>
        </b-card>
      </div>
    </b-card-body>
    <b-card-footer>

    </b-card-footer>
    <ecosystem-modal v-on:selection="updateEcosystem" />
    <vuln-source-c-s-a-f-add />
    <vuln-source-c-s-a-f-compare />
    <vuln-source-c-s-a-f-upload />
  </b-card>
</template>
<script>
import { Switch as cSwitch } from '@coreui/vue';
import common from '../../../shared/common';
import configPropertyMixin from '../mixins/configPropertyMixin';
import EcosystemModal from './EcosystemModal';
import VulnSourceCSAFAdd from './VulnSourceCSAFAddModal.vue';
import ActionableListGroupItem from '../../components/ActionableListGroupItem.vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import VulnSourceCSAFCompare from './VulnSourceCSAFCompare.vue'
import VulnSourceCSAFUpload from './VulnSourceCSAFUpload.vue'

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String,
  },
  components: {
    cSwitch,
    EcosystemModal,
    VulnSourceCSAFAdd,
    ActionableListGroupItem,
    BValidatedInputGroupFormInput,
    VulnSourceCSAFCompare,
    VulnSourceCSAFUpload,
  },
  data() {
    return {
      selectedRows: [],
      vulnsourceEnabled: false,
      aliasSyncEnabled: false,
      //aliasGitHubIssueUrl: 'https://github.com/google/osv.dev/issues/888',
      csafBaseUrl:
        'https://wid.cert-bund.de/.well-known/csaf-aggregator/aggregator.json', //TODO provide through system settings
      ecosystemConfig: null,
      enabledEcosystems: [],
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      srcCols: [
        {
          title: 'ID',
          field: 'csafEntryId',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,
        },
        {
          title: this.$t('admin.enabled'),
          field: 'enabled',
          class: 'tight',
          sortable: true,
          formatter(value, row, index) {
            return value === true ? '<i class="fa fa-check-square-o" />' : '';
          },
        },
      ],
      srcData: [],
      srcOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
        silentSort: false,
        icons: {
          refresh: 'fa-refresh',
        },
      },
    };
  },
  watch: {
    vulnsourceEnabled(toggleChange) {
      if (toggleChange === false) {
        this.enabledEcosystems = [];
        this.updateConfigProperties([
          {
            groupName: 'vuln-source',
            propertyName: 'google.osv.enabled',
            propertyValue: null,
          },
        ]);
      }
    },
  },
  methods: {
    handleAdd(id) {
      const row = this.sampleRecData.find(item => item.id === id);
      console.log('Row added:', row);
      // TODO: Add to sources
    },
    onCheckChange(selected){
      this.selectedRows = selected;
      console.log("selected");
      console.log(this.selectedRows);
    },
    openCompare() {
      if (this.selectedRows.length !== 2) {  //debugging, should be "==="
        this.$bvModal.show('vulnSourceCSAFCompareModal');
      } else {
        alert(this.$t('admin.please_select_two_rows'));
      }
    },
    apiUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_ENTITY}`;
    },
    refreshCsafSourcesTable: function () {
      this.$refs.table_sources.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    removeEcosystem: function (ecosystem) {
      this.enabledEcosystems = this.enabledEcosystems.filter(
        (e) => e !== ecosystem,
      );
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    updateEcosystem: function (ecosystems) {
      this.$root.$emit('bv::hide::modal', 'ecosystemModal');
      for (let i = 0; i < ecosystems.length; i++) {
        let ecosystem = ecosystems[i];
        this.enabledEcosystems.push(ecosystem.name);
      }
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    saveUrl: function () {
      /*
      this.updateConfigProperties([
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.base.url',
          propertyValue: this.osvBaseUrl,
        },
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.enabled',
          propertyValue: this.enabledEcosystems.join(';'),
        },
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.alias.sync.enabled',
          propertyValue: this.aliasSyncEnabled,
        },
      ]);*/
    },
    updateSourcesTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        console.log(response);
        this.srcData = response;

        this.$toastr.s('Csaf sources updated');
      });
    },
  },
  created() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'vuln-source';
      });
      this.refreshCsafSourcesTable();
      this.enabledEcosystems = this.ecosystemConfig
        .split(';')
        .map((ecosystem) => ecosystem.trim());
    });
  },
};
</script>
