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
                  :columns="sampleSrcColumns"
                  :data="sampleSrcData"
                  :options="sampleSrcOpts"
                >
                </bootstrap-table>
                <h3>{{ $t('admin.suggested_discovery_sources') }}:</h3>
                <bootstrap-table
                  ref="table_suggested"
                  :columns="sampleRecColumns"
                  :data="sampleRecData"
                  :options="sampleRecOpts"
                >
                </bootstrap-table>
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
                  :columns="sampleDocColumns"
                  :data="sampleDocData"
                  :options="sampleDocOpts"
                  data-click-to-select="true"
                  @check-change="onCheckChange"
                >
                </bootstrap-table>
              </b-card-body>
            </b-tab>
          </b-tabs>
        </b-card>
      </div>
      <!--<b-validated-input-group-form-input
        id="csaf-feeds-url"
        :label="$t('admin.vulnsource_csaf_base_url')"
        input-group-size="mb-3"
        rules="required"
        type="text"
        v-model="csafBaseUrl"
        lazy="true"
      />-->

      <hr />
      <b-form-group label="Ecosystems">
        <div class="list-group" style="width: 40%">
          <span v-for="ecosystem in enabledEcosystems" :key="ecosystem">
            <actionable-list-group-item
              :value="ecosystem"
              :delete-icon="true"
              @actionClicked="removeEcosystem(ecosystem)"
            />
          </span>
          <actionable-list-group-item
            :add-icon="true"
            @actionClicked="$root.$emit('bv::show::modal', 'ecosystemModal')"
          />
        </div>
      </b-form-group>
    </b-card-body>
    <b-card-footer>
      <b-button
        :disabled="this.vulnsourceEnabled && !this.csafBaseUrl"
        variant="outline-primary"
        class="px-4"
        @click="saveUrl"
      >
        {{ $t('message.update') }}
      </b-button>
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
import VulnSourceCSAFAdd from './VulnSourceCSAFAdd.vue'
import VulnSourceCSAFCompare from './VulnSourceCSAFCompare.vue'
import VulnSourceCSAFUpload from './VulnSourceCSAFUpload.vue'
import ActionableListGroupItem from '../../components/ActionableListGroupItem.vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String,
  },
  components: {
    cSwitch,
    EcosystemModal,
    VulnSourceCSAFAdd,
    VulnSourceCSAFCompare,
    VulnSourceCSAFUpload,
    ActionableListGroupItem,
    BValidatedInputGroupFormInput,
  },
  data() {
    return {
      vulnsourceEnabled: false,
      aliasSyncEnabled: false,
      aliasGitHubIssueUrl: 'https://github.com/google/osv.dev/issues/888',
      csafBaseUrl:
        'https://wid.cert-bund.de/.well-known/csaf-aggregator/aggregator.json', //TODO provide through system settings
      ecosystemConfig: null,
      enabledEcosystems: [],
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      // Sample Data for CSAF Sources:
      sampleSrcColumns: [
        {
          title: 'ID',
          field: 'id',
          sortable: true,

        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,

        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,

        },
      ],
      sampleSrcData: [
        { id: 1, name: 'BSI WID', url: 'https://wid.cert-bund.de/.well-known/csaf-aggregator/aggregator.json' },
        { id: 2, name: 'Red Hat', url: 'https://www.redhat.com/.well-known/csaf/provider-metadata.json' },
        { id: 3, name: 'Example service', url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.json' },
      ],
      sampleSrcOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
      },
      // Sample Data for recommended sources
      sampleRecColumns: [
        {
          title: 'ID', //this.$t('admin.identifier'),
          field: 'id',
          sortable: true,

        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,

        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,

        },
        {
          title: 'New',
          field: 'new',
          class: 'tight',
          sortable: true,

        },
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" onclick="handleAdd(${row.id})" >  <span class="fa fa-plus"></span> Add</button>`;
          },
        },
      ],
      sampleRecData: [
        { id: 1, name: 'Example service 2', url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.json', new: 'New'},
        { id: 2, name: 'Example service 3', url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.json'},
      ],
      sampleRecOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
      },
      // Sample Data for CASF Documents
      sampleDocColumns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          title: 'ID', //this.$t('admin.identifier'),
          field: 'id',
          sortable: true,

        },
        {
          title: 'Vendor',
          field: 'vendor',
          sortable: true,

        },
        {
          title: 'Version',
          field: 'version',
          class: 'tight',
          sortable: true,

        },
        {
          title: 'Last updated',
          field: 'last_updated',
          class: 'tight',
          sortable: true,

        },
      ],
      sampleDocData: [
        { id: 1, vendor: 'Vendor 1', version: '1.0', last_updated: '12-07-24'},
        { id: 2, vendor: 'Vendor 2', version: '1.3', last_updated: '06-03-24'},
      ],
      sampleDocOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,

      },
      selectedRows: [],
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
  },
  created() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'vuln-source';
      });
      /*for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'google.osv.enabled':
            this.ecosystemConfig = item.propertyValue;
            this.vulnsourceEnabled = this.ecosystemConfig != null;
            break;
          case 'google.osv.alias.sync.enabled':
            this.aliasSyncEnabled = common.toBoolean(item.propertyValue);
            break;
          case 'google.osv.base.url':
            this.osvBaseUrl = item.propertyValue;
            break;
        }
      }*/
      this.enabledEcosystems = this.ecosystemConfig
        .split(';')
        .map((ecosystem) => ecosystem.trim());
    });
  },
};
</script>
