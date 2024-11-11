<template>
  <b-card no-body :header="header">
    <b-card-body>
      <img alt="CSAF logo" src="@/assets/img/csaf-logo.png" width="130" />
      <hr />
      <c-switch
        color="primary"
        id="vulnsourceEnabled"
        label
        v-bind="labelIcon"
        v-model="vulnsourceEnabled"
        :disabled="!configInitialized"
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
                <!--<h3>{{ $t('admin.csaf_sources') }}:</h3>-->
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
                <hr />
              </b-card-body>
              <!--<repository-create-repository-modal
                  :type="type"
                  v-on:refreshTable="refreshTable"
                />-->
              <!--</b-card>-->
            </b-tab>
            <b-tab title="CSAF Documents">
              <b-card-body>
                <!--<h2>{{ $t('admin.csaf_documents') }}:</h2>-->
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
    <b-card-footer></b-card-footer>
    <ecosystem-modal v-on:selection="updateEcosystem" />
    <vuln-source-c-s-a-f-add v-on:refreshTable="refreshCsafSourcesTable" />
    <vuln-source-c-s-a-f-compare />
    <vuln-source-c-s-a-f-upload />
  </b-card>
</template>
<script>
import { Switch as cSwitch } from '@coreui/vue';
import configPropertyMixin from '../mixins/configPropertyMixin';
import EcosystemModal from './EcosystemModal';
import VulnSourceCSAFAdd from './VulnSourceCSAFAddModal.vue';
import ActionableListGroupItem from '../../components/ActionableListGroupItem.vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import VulnSourceCSAFCompare from './VulnSourceCSAFCompare.vue';
import VulnSourceCSAFUpload from './VulnSourceCSAFUpload.vue';
import i18n from '../../../i18n';
import bootstrapTableMixin from '../../../mixins/bootstrapTableMixin';
import EventBus from '../../../shared/eventbus';

export default {
  mixins: [configPropertyMixin, bootstrapTableMixin],
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
      configInitialized: false, // Wait for retrieving config
      vulnsourceEnabled: false,
      vulnsourceToggleInitialized: false,
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
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        icons: {
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: false,
        detailViewByClick: true,
        onExpandRow: this.vueFormatterInit,
        detailFormatter: (index, row) => {
          return this.vueFormatter({
            i18n,
            template: `
                <b-row class="expanded-row">
                  <b-col sm="6">
                    <b-validated-input-group-form-input
                      id="url" :label="$t('admin.url')"
                      input-group-size="mb-3" rules="required"
                      type="url" v-model="url"
                      autofocus="true"
                      v-debounce:750ms="updateCsafSource" :debounce-events="'keyup'"/>
                  </b-col>
                  <b-col sm="6">

                    <div>
                      <c-switch color="primary" v-model="internal" label v-bind="labelIcon" />{{$t('admin.internal')}}
                    </div>
                    <div>
                     <c-switch color="primary" v-model="authenticationRequired" label v-bind="labelIcon" />{{$t('admin.repository_authentication')}}
                    </div>

                    <div>
                      <b-validated-input-group-form-input
                        id="username" :label="$t('admin.username')"
                        input-group-size="mb-3"
                        v-model="username"
                        rules="required"
                        v-show="authenticationRequired"
                        v-debounce:750ms="updateCsafSource" :debounce-events="'keyup'"/>
                    </div>

                    <div>
                      <b-validated-input-group-form-input
                        id="password" :label="$t('admin.password')"
                        input-group-size="mb-3"
                        type="password"
                        v-model="password"
                        rules="required"
                        v-show="authenticationRequired"
                        v-debounce:750ms="updateCsafSource" :debounce-events="'keyup'"/>
                    </div>

                    <div>
                      <c-switch color="primary" v-model="enabled" label v-bind="labelIcon" />{{$t('admin.enabled')}}
                    </div>

                    <div style="text-align:right">
                       <b-button variant="outline-danger" @click="deleteCsafSource">Delete CSAF source</b-button>
                    </div>
                  </b-col>
                </b-row>
            `,
            components: {
              cSwitch,
              BValidatedInputGroupFormInput,
            },
            data() {
              return {
                csafEntry: row,
                csafEntryId: row.csafEntryId,
                name: row.name,
                url: row.url,
                enabled: row.enabled,
                labelIcon: {
                  dataOn: '\u2713',
                  dataOff: '\u2715',
                },
              };
            },
            watch: {
              enabled() {
                this.updateCsafSource();
              },
            },
            methods: {
              deleteCsafSource: function () {
                console.log(`delete csaf source ${this.csafEntryId}`);
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_ENTITY}/${this.csafEntryId}`;
                this.axios
                  .delete(url)
                  .then((response) => {
                    EventBus.$emit('admin:csafSources:rowDeleted', index);
                    this.$toastr.s(this.$t('admin.repository_deleted'));
                  })
                  .catch((error) => {
                    this.$toastr.w(this.$t('condition.unsuccessful_action'));
                  });
              },
              updateCsafSource: function () {
                console.log(`update entry ${this.csafEntryId}`);
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_ENTITY}`;
                this.axios
                  .post(url, {
                    csafEntryId: this.csafEntryId,
                    url: this.url,
                    name: this.name,
                    //authenticationRequired: this.authenticationRequired,
                    //username: this.username,
                    //password:
                    //  this.password || 'HiddenDecryptedPropertyPlaceholder',
                    enabled: this.enabled,
                    //uuid: this.uuid,
                  })
                  .then((response) => {
                    this.csafEntry = response.data;
                    EventBus.$emit(
                      'admin:csafSources:rowUpdate',
                      index,
                      this.csafEntry,
                    );
                    this.$toastr.s(this.$t('message.updated'));
                  })
                  .catch((error) => {
                    this.$toastr.w(this.$t('condition.unsuccessful_action'));
                  });
              },
            }
          });
        },
      },
    };
  },
  watch: {
    vulnsourceEnabled(newValue) {
      if (!this.vulnsourceToggleInitialized) {
        this.vulnsourceToggleInitialized = true; // skip when initializing
      } else {
        this.updateConfigProperties([
          {
            groupName: 'vuln-source',
            propertyName: 'csaf.enabled',
            propertyValue: this.vulnsourceEnabled,
          },
        ]);
      }
    },
  },
  methods: {
    handleAdd(id) {
      const row = this.sampleRecData.find((item) => item.id === id);
      console.log('Row added:', row);
      // TODO: Add to sources
    },
    onCheckChange(selected) {
      this.selectedRows = selected;
      console.log('selected');
      console.log(this.selectedRows);
    },
    openCompare() {
      if (this.selectedRows.length !== 2) {
        //debugging, should be "==="
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
    saveConfiguration: function () {
      console.log('Saving config changes ');
      this.updateConfigProperties([
        {
          groupName: 'vuln-source',
          propertyName: 'csaf.enabled',
          propertyValue: this.vulnsourceEnabled,
        },
      ]);
    },
    updateSourcesTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        console.log(response);
        this.srcData = response;

        this.$toastr.s('Csaf sources updated');
      });
    },
  },
  mounted() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'vuln-source';
      });
      for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'csaf.enabled':
            if (item.propertyValue === 'true') {
              this.vulnsourceEnabled = true;
            } else {
              this.vulnsourceToggleInitialized = true; // toggle is initialized
            }
            break;
        }
      }
      this.refreshCsafSourcesTable();
      this.configInitialized = true;
    });
    EventBus.$on('admin:csafSources:rowDeleted', (index, row) => {
      this.refreshCsafSourcesTable();
    });
    EventBus.$on('admin:csafSources:rowUpdate', (index, row) => {
      this.$refs.table_sources.updateRow({ index: index, row: row });
      this.$refs.table_sources.expandRow(index);
    });
  },
  beforeDestroy() {
    EventBus.$off('admin:csafSources:rowUpdated');
    EventBus.$off('admin:csafSources:rowDeleted');
  }
};
</script>
