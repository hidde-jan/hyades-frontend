<template>
  <b-card no-body :header="header">
    <b-card-body>
      <h1>CSAF</h1>
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
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="triggerAll"
                  >
                    <span class="fa fa-refresh"></span>
                    {{ $t('admin.trigger_all') }}
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
                  :columns="recColumns"
                  :data="recData"
                  :options="recOptions"
                  data-click-to-select="true"
                >
                </bootstrap-table>
                <b-button size="md" variant="outline-primary" @click="markReadSuggestions">
                  <span class="fa fa-check"></span>
                  {{ $t('admin.mark_selected_read') }}
                </b-button>
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
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                <!--<h2>{{ $t('admin.csaf_documents') }}:</h2>-->
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="openCompare"
                >
                  <span class="fa fa-file"></span>
                  {{ $t('admin.compare_selected') }}
                </b-button>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="deleteSelected"
                >
                  <span class="fa fa-trash"></span>
                  {{ $t('admin.delete_selected') }}
                </b-button>
                  <b-button size="md" variant="outline-primary" @click="markReadDocuments">
                    <span class="fa fa-check"></span>
                    {{ $t('admin.mark_selected_read') }}
                  </b-button>
                </div>
                <bootstrap-table
                  ref="table_documents"
                  :columns="docColumns"
                  :data="docData"
                  :options="docOpts"
                  data-click-to-select="true"
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
    <vuln-source-c-s-a-f-compare
      :leftTitle="compareLeftTitle"
      :rightTitle="compareRightTitle"
      :leftContent="compareLeftContent"
      :rightContent="compareRightContent"
    />
    <vuln-source-c-s-a-f-upload />
    <vuln-source-c-s-a-f-view-doc-modal
      :title="detailTitle"
      :content="detailContent"
    />
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
import VulnSourceCSAFViewDocModal from './VulnSourceCSAFViewDocModal.vue';
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
    VulnSourceCSAFViewDocModal,
  },
  data() {
    return {
      compareLeftTitle: '',
      compareLeftContent: '',
      compareRightTitle: '',
      compareRightContent: '',
      detailTitle: '',
      detailContent: '',
      configInitialized: false, // Wait for retrieving config
      vulnsourceEnabled: false,
      vulnsourceToggleInitialized: false,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      recColumns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          title: 'ID',
          field: 'csafEntryId',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter: (value, row) => {
            return row.new === 'New' ? `${value} *` : value;
          },
        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,
          width: '350px',
        },
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="rec-${row.csafEntryId}" >  <span class="fa fa-plus"></span> Add</button>`;
          },
        },
      ],
      recData: [
        //TODO: Remove when real data available
        // test data:
        //{ id: 1, name: 'Example service 2', url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.json', new: 'New'},
        //{ id: 2, name: 'Example service 3', url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.json'},
      ],
      recOptions: {
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
        rowAttributes: (row) => {
          return {
            style: row.new === 'New' ? 'font-weight: bold;' : '',
          };
        },
      },
      docColumns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          title: 'ID',
          field: 'csafEntryId',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
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
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="doc-${row.csafEntryId}"> <span class="fa fa-search-plus"></span> View Details</button>`;
          },
        },
      ],
      docData: [
        // for tests, delete later
        //{ csafEntryId: 1, vendor: 'Vendor 1', version: '1.0', last_updated: '12-07-24'},
        //{ csafEntryId: 2, vendor: 'Vendor 2', version: '1.3', last_updated: '06-03-24'},
      ],
      docOpts: {
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
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}/${this.csafEntryId}`;
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
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;
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
            },
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
    showDoc(docId) {
      console.log('docData:', this.docData);
      console.log('ID:', docId);
      const row = this.docData.find((item) => item.csafEntryId === docId);
      console.log('Show doc:', row);
      this.detailTitle = row.name;
      this.detailContent = this.getDocument(docId);
      //for tests
      //this.detailTitle = 'test';
      //this.detailContent = 'some content';
      this.$bvModal.show('vulnSourceCSAFViewDocModal');
    },
    handleAdd(id) {
      console.log('recData:', this.srcData);
      console.log(`id clicked: ${id}`);
      const row = this.recData.find((item) => item.csafEntryId.toString() === id.toString());
      console.log('Row added:', row);
      // TODO: Add clicked suggested source to sources
      // TODO: Remove clicked suggested source from suggested
      //this.refreshCsafSourcesTable();
      //this.refreshCsafSuggestedTable();
    },
    openCompare() {
      const selectedRows = this.$refs.table_documents.getSelections();
      console.log('Selected rows:', selectedRows);
      this.compareLeftTitle = selectedRows[0].name;
      this.compareLeftContent = this.getDocument(selectedRows[0].csafEntryId);
      this.compareRightTitle = selectedRows[1].name;
      this.compareRightContent = this.getDocument(selectedRows[1].csafEntryId);
      // for tests:
      //this.compareLeftTitle = 'CSAF 1';
      //this.compareLeftContent = 'CSAF CSAF CSAF';
      //this.compareRightTitle = 'CSAF 2';
      //this.compareRightContent = 'C$AF CSAF CSAF';
      if (selectedRows.length === 2) {
        this.$bvModal.show('vulnSourceCSAFCompareModal');
      } else {
        alert(this.$t('admin.please_select_two_rows'));
      }
    },
    deleteSelected() {
      const selectedRows = this.$refs.table_documents.getSelections();

      if (selectedRows.length === 0) {
        console.log(`nothing selected`);
        return;
      }

      const deletePromises = selectedRows.map((row) => {
        const url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}/${row.csafEntryId}`;
        return this.axios
          .delete(url)
          .then((response) => {
            EventBus.$emit('admin:csafDocuments:rowDeleted', row.id);
            this.$toastr.s(this.$t('admin.document_deleted', { id: row.id }));
          })
          .catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
      });

      Promise.all(deletePromises).then(() => {
        this.refreshCsafDocumentsTable();
      });
    },
    markReadSuggestions() {
      const selectedRows = this.$refs.table_suggested.getSelections();
      console.log('Read:', selectedRows);
      //TODO: mark as read, which api endpoint?
      //TODO: refresh tables
    },
    markReadDocuments() {
      const selectedRows = this.$refs.table_documents.getSelections();
      console.log('Read:', selectedRows);
      //TODO: mark as read, which api endpoint?
      //TODO: refresh tables
    },
    triggerAll() {
      console.log('Trigger all');
      const url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_TRIGGER}/`;
      return this.axios
        .post(url)
        .then((response) => {
          this.$toastr.s(this.$t('admin.trigger_all'));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    getDocument(docId) {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}/${docId}`;
      this.axios
        .get(url)
        .then((response) => {
          this.$toastr.s(this.$t('admin.repository_deleted'));
          return response.toString();
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    apiUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;
    },
    apiDocsUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}`;
    },
    apiDisUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DISCOVERY}`;
    },
    refreshCsafSourcesTable: function () {
      this.$refs.table_sources.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    refreshCsafSuggestedTable: function () {
      this.$refs.table_suggested.refresh({
        url: this.apiDisUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    refreshCsafDocumentsTable: function () {
      this.$refs.table_documents.refresh({
        url: this.apiDocsUrl(),
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
    //TODO: find correct endpoint
    updateDocumentsTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        console.log(response);
        this.docData = response;

        this.$toastr.s('Csaf documents updated');
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
      this.refreshCsafDocumentsTable();
      this.refreshCsafSuggestedTable();
      this.configInitialized = true;
    });
    EventBus.$on('admin:csafSources:rowDeleted', (index, row) => {
      this.refreshCsafSourcesTable();
    });
    EventBus.$on('admin:csafSources:rowUpdate', (index, row) => {
      this.$refs.table_sources.updateRow({ index: index, row: row });
      this.$refs.table_sources.expandRow(index);
    });
    this.$refs.table_documents.$el.addEventListener('click', (event) => {
      const target = event.target;
      if (target.matches('[id^="doc-"]')) {
        const docId = target.id.split('-')[1];
        this.showDoc(docId);
      }
    });
    this.$refs.table_suggested.$el.addEventListener('click', (event) => {
      const target = event.target;
      if (target.matches('[id^="rec-"]')) {
        const recId = target.id.split('-')[1];
        this.handleAdd(recId);
      }
    });
  },
  beforeDestroy() {
    EventBus.$off('admin:csafSources:rowUpdated');
    EventBus.$off('admin:csafSources:rowDeleted');
  },
};
</script>
