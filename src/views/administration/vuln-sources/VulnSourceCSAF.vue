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
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.csaf_aggregators') }}</h4>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="
                    $root.$emit('setModalTitle', $t('admin.add_aggregator'))
                  "
                  v-b-modal.vulnSourceCSAFAddModal
                >
                  <span class="fa fa-plus"></span>
                  {{ $t('admin.add_aggregator') }}
                </b-button>
                <bootstrap-table
                  ref="table_sources"
                  :columns="srcCols"
                  :data="srcData"
                  :options="srcOpts"
                >
                </bootstrap-table>
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.csaf_providers') }}</h4>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="
                    $root.$emit('setModalTitle', $t('admin.add_provider'))
                  "
                  v-b-modal.vulnSourceCSAFAddModal
                >
                  <span class="fa fa-plus"></span>
                  {{ $t('admin.add_provider') }}
                </b-button>
                <bootstrap-table
                  ref="table_providers"
                  :columns="provCols"
                  :data="provData"
                  :options="provOpts"
                >
                </bootstrap-table>
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.suggested_discovery_sources') }}</h4>
                <bootstrap-table
                  ref="table_suggested"
                  :columns="recColumns"
                  :data="recData"
                  :options="recOptions"
                  data-click-to-select="true"
                >
                </bootstrap-table>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="markReadSuggestions"
                  >
                    <span class="fa fa-check"></span>
                    {{ $t('admin.mark_selected_read') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="addSelected"
                  >
                    <span class="fa fa-plus"></span>
                    {{ $t('admin.add_selected') }}
                  </b-button>
                </div>
                <hr />
              </b-card-body>
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
                </div>
                <bootstrap-table
                  ref="table_documents"
                  :columns="docColumns"
                  :data="docData"
                  :options="docOpts"
                  data-click-to-select="true"
                >
                </bootstrap-table>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="markReadDocuments"
                  >
                    <span class="fa fa-check"></span>
                    {{ $t('admin.mark_selected_read') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="deleteSelected"
                  >
                    <span class="fa fa-trash"></span>
                    {{ $t('admin.delete_selected') }}
                  </b-button>
                </div>
              </b-card-body>
            </b-tab>
          </b-tabs>
        </b-card>
      </div>
    </b-card-body>
    <b-card-footer></b-card-footer>
    <ecosystem-modal v-on:selection="updateEcosystem" />
    <vuln-source-c-s-a-f-add v-on:refreshTable="refreshBothCsafSourcesTables" />
    <vuln-source-c-s-a-f-compare
      :leftTitle="compareLeftTitle"
      :rightTitle="compareRightTitle"
      :leftContent="compareLeftContent"
      :rightContent="compareRightContent"
    />
    <vuln-source-c-s-a-f-upload />
    <vuln-source-c-s-a-f-permission />
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
import VulnSourceCSAFPermission from './VulnSourceCSAFPermission.vue';
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
    VulnSourceCSAFPermission,
  },
  data() {
    return {
      compareLeftTitle: '',
      compareLeftContent: null,
      compareRightTitle: '',
      compareRightContent: null,
      detailTitle: '',
      detailContent: null,
      searchTerm: '',
      configInitialized: false,
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
          field: 'id',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter: (value, row) => {
            return row.seen === false ? `<strong>${value} *</strong>` : value;
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
          title: 'Read',
          field: 'seen',
          sortable: true,
        },
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="rec-${row.id}" >  <span class="fa fa-plus"></span> Add</button>`;
          },
        },
      ],
      recData: [],
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
        sortName: 'seen',
        sortOrder: 'asc',
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
          field: 'id',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter: (value, row) => {
            return row.seen === false ? `<strong>${value} *</strong>` : value;
          },
        },
        {
          title: 'Publisher',
          field: 'publisherNamespace',
          sortable: true,
        },
        {
          title: 'Version',
          field: 'trackingVersion',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Last fetched',
          field: 'lastFetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value * 1000);
            return date.toLocaleString();
          },
        },
        {
          title: 'Read',
          field: 'seen',
          sortable: true,
        },
        {
          title: 'Newest Version?',
          field: 'latestVersion',
          formatter: (value, row) => {
            const allRows = this.$refs.table_documents.getData();
            const sameNameDocs = allRows.filter((doc) => doc.name === row.name);
            const maxVersion = Math.max(
              ...sameNameDocs.map(
                (doc) => Number(doc.trackingVersion) || -Infinity,
              ),
            );
            return Number(row.trackingVersion) === maxVersion ? 'Yes' : 'No';
          },
        },
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="doc-${row.id}"> <span class="fa fa-search-plus"></span> View Details</button>
                    <button class="btn btn-primary" id="reanalyze-${row.id}"> <span class="fa fa-refresh"></span> Reanalyze </button>
`;
          },
        },
      ],
      docData: [],
      docOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'server',
        dataField: 'objects',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'seen',
        sortOrder: 'asc',
        icons: {
          refresh: 'fa-refresh',
        },
      },
      srcCols: [
        {
          title: 'ID',
          field: 'id',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          class: 'tight',
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
        },
        {
          title: 'Last fetched',
          field: 'lastFetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value * 1000);
            return date.toLocaleString();
          },
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
        onLoadError: (status, res) => {
          if (status === 403) {
            this.$bvModal.show('vulnSourceCSAFPermission');
          }
        },
        detailFormatter: (index, row) => {
          return this.vueFormatter({
            i18n,
            template: `
                <b-row class="expanded-row">
                  <b-col sm="6">
                    <b-validated-input-group-form-input
                      id="surl" :label="$t('admin.url')"
                      input-group-size="mb-3" rules="required|domainOrUrl"
                      type="url" v-model="surl"
                      autofocus="true"
                    />
                  </b-col>
                  <b-col sm="6">
                    <div>
                      <c-switch color="primary" v-model="enabled" label v-bind="labelIcon" />{{$t('admin.enabled')}}
                    </div>
                    <div style="text-align:right">
                       <b-button variant="outline-danger" @click="deleteCsafSource">Delete CSAF source</b-button>
                       <b-button variant="outline-danger" @click="resetFetched">Reset last fetched</b-button>
                       <b-button variant="outline-primary" @click="updateCsafSource">Save changes</b-button>
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
                sid: row.id,
                name: row.name,
                surl: row.url,
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
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}/${this.sid}`;
                this.axios
                  .delete(url)
                  .then((response) => {
                    EventBus.$emit('admin:csafAggregators:rowDeleted', index);
                    this.$toastr.s(this.$t('admin.csaf_source_deleted'));
                  })
                  .catch((error) => {
                    this.$toastr.w(this.$t('condition.unsuccessful_action'));
                  });
              },
              resetFetched() {
                this.lastFetched = null;
                this.updateCsafSource().then(() => {
                  EventBus.$emit('refreshAggregatorsTable');
                });
              },
              updateCsafSource: function () {
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;
                return this.axios
                  .post(url, {
                    id: this.sid,
                    url: this.surl,
                    name: this.name,
                    enabled: this.enabled,
                  })
                  .then((response) => {
                    this.csafEntry = response.data;
                    EventBus.$emit(
                      'admin:csafAggregators:rowUpdate',
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
      provCols: [
        {
          title: 'ID',
          field: 'id',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          class: 'tight',
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
        },
        {
          title: 'Last fetched',
          field: 'lastFetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value * 1000);
            return date.toLocaleString();
          },
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
      provData: [],
      provOpts: {
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
                      input-group-size="mb-3" rules="required|domainOrUrl"
                      type="url" v-model="url"
                      autofocus="true"
                    />
                  </b-col>
                  <b-col sm="6">
                    <div>
                      <c-switch color="primary" v-model="enabled" label v-bind="labelIcon" />{{$t('admin.enabled')}}
                    </div>
                    <div style="text-align:right">
                      <b-button variant="outline-danger" @click="deleteCsafSource">Delete CSAF source</b-button>
                      <b-button variant="outline-danger" @click="resetFetched">Reset last fetched</b-button>
                      <b-button variant="outline-primary" @click="updateCsafSource">Save changes</b-button>
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
                id: row.id,
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
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}/${this.id}`;
                this.axios
                  .delete(url)
                  .then((response) => {
                    EventBus.$emit('admin:csafProviders:rowDeleted', index);
                    this.$toastr.s(this.$t('admin.csaf_source_deleted'));
                  })
                  .catch((error) => {
                    this.$toastr.w(this.$t('condition.unsuccessful_action'));
                  });
              },
              resetFetched() {
                this.lastFetched = null;
                this.updateCsafSource().then(() => {
                  EventBus.$emit('refreshProvidersTable');
                });
              },
              updateCsafSource: function () {
                let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_PROVIDER}`;
                return this.axios
                  .post(url, {
                    id: this.id,
                    url: this.url,
                    name: this.name,
                    enabled: this.enabled,
                  })
                  .then((response) => {
                    this.csafEntry = response.data;
                    EventBus.$emit(
                      'admin:csafProviders:rowUpdate',
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
    async showDoc(docId) {
      const srow = this.$refs.table_documents
        .getData()
        .find((item) => item.id.toString() === docId.toString());
      this.detailTitle = srow.name;
      this.detailContent = await this.getDocument(docId);
      this.$bvModal.show('vulnSourceCSAFViewDocModal');
    },
    reanalyze(docId) {
      console.log(`Reanalyzing document with ID: ${docId}`);
      //TODO: Add api call
    },
    handleAdd(id) {
      var addRow = this.$refs.table_suggested
        .getData()
        .find((item) => item.id.toString() === id.toString());
      addRow.discovery = false;
      this.updateCsafSource(addRow);
    },
    addSelected() {
      const selectedRows = this.$refs.table_suggested.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((item) => {
          item.discovery = false;
          this.updateCsafSource(item);
        });
      }
    },
    async updateCsafSource(prow) {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_PROVIDER}`;
      try {
        const response = await this.axios.post(url, {
          id: prow.id,
          url: prow.url,
          name: prow.name,
          discovery: prow.discovery,
          enabled: prow.enabled,
          fetchInterval: prow.fetchInterval,
          aggregator: prow.aggregator,
          seen: prow.seen,
        });
        this.csafEntry = response.data;
        EventBus.$emit(
          'admin:csafProviders:rowUpdate',
          prow.id,
          this.csafEntry,
        );
        this.$toastr.s(this.$t('message.updated'));
        this.refreshCsafSuggestedTable();
        this.refreshProvidersTable(); // TODO re-evaluate necessity with rowUpdate
      } catch (error) {
        this.$toastr.w(this.$t('condition.unsuccessful_action'));
      }
    },
    async openCompare() {
      const selectedRows = this.$refs.table_documents.getSelections();
      this.compareLeftTitle = selectedRows[0].name;
      this.compareLeftContent = await this.getDocument(selectedRows[0].id);
      this.compareRightTitle = selectedRows[1].name;
      this.compareRightContent = await this.getDocument(selectedRows[1].id);
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
        var rowIndex = this.$refs.table_documents
          .getData()
          .findIndex((item) => item.id === row.id);
        const url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}/${row.id}`;
        return this.axios
          .delete(url)
          .then((response) => {
            EventBus.$emit('admin:csafDocuments:rowDeleted', rowIndex);
            this.$toastr.s(
              this.$t('admin.csaf_document_deleted', { id: row.id }),
            );
          })
          .catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
      });
    },
    markReadSuggestions() {
      const selectedRows = this.$refs.table_suggested.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((prow) => {
          prow.seen = true;
          this.updateCsafSource(prow); // TODO issue update on eventbus
        });
      }
      this.refreshCsafSuggestedTable();
    },
    markReadDocuments() {
      const selectedRows = this.$refs.table_documents.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((row) => {
          // find row index
          var rowIndex = this.$refs.table_documents
            .getData()
            .findIndex((item) => item.id === row.id);

          let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}/seen/${row.id}`;
          this.axios
            .post(url, { id: row.id })
            .then((response) => {
              EventBus.$emit(
                'admin:csafDocuments:rowUpdate',
                rowIndex,
                response.data,
              );
              this.$toastr.s(this.$t('admin.csaf_document_read'));
            })
            .catch(() => {
              this.$toastr.w(this.$t('condition.unsuccessful_action'));
            });
        });
      }
    },
    triggerAll() {
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
      return this.axios
        .get(url)
        .then((response) => {
          return response.data;
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
          return null;
        });
    },
    apiUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;
    },
    apiProvidersUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_PROVIDER}`;
    },
    apiDocsUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}`;
    },
    apiDisUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DISCOVERY}`;
    },
    refreshBothCsafSourcesTables: function () {
      this.refreshAggregatorsTable();
      this.refreshProvidersTable();
    },
    refreshAggregatorsTable: function () {
      this.$refs.table_sources.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    refreshProvidersTable: function () {
      this.$refs.table_providers.refresh({
        url: this.apiProvidersUrl(),
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
        this.srcData = response;

        this.$toastr.s('Csaf sources updated');
      });
    },
    updateProvidersTable: function () {
      this.axios.get(this.apiProvidersUrl()).then((response) => {
        this.provData = response;
        this.$toastr.s('Csaf providers updated');
      });
    },
    updateDocumentsTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
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
      this.refreshBothCsafSourcesTables();
      this.refreshCsafDocumentsTable();
      this.refreshCsafSuggestedTable();
      this.configInitialized = true;
    });
    EventBus.$on('admin:csafAggregators:rowDeleted', (index, row) => {
      this.refreshAggregatorsTable();
    });
    EventBus.$on('admin:csafAggregators:rowUpdate', (index, row) => {
      this.$refs.table_sources.updateRow({ index: index, row: row });
      this.$refs.table_sources.expandRow(index);
    });
    EventBus.$on('admin:csafProviders:rowDeleted', (index, row) => {
      this.refreshProvidersTable();
    });
    EventBus.$on('admin:csafProviders:rowUpdate', (index, row) => {
      this.$refs.table_providers.updateRow({ index: index, row: row });
      this.$refs.table_providers.expandRow(index);
    });
    EventBus.$on('admin:csafDocuments:rowUpdate', (index, row) => {
      this.$refs.table_documents.updateRow({ index: index, row: row });
    });
    EventBus.$on('admin:csafDocuments:rowDeleted', (index, row) => {
      this.refreshCsafDocumentsTable();
    });
    EventBus.$on('refreshProvidersTable', () => {
      this.refreshProvidersTable();
    });
    EventBus.$on('refreshAggregatorsTable', () => {
      this.refreshAggregatorsTable();
    });
    this.$refs.table_documents.$el.addEventListener('click', (event) => {
      const target = event.target;
      if (target.matches('[id^="doc-"]')) {
        const docId = target.id.split('-')[1];
        this.showDoc(docId);
      }
      if (target.matches('[id^="reanalyze-"]')) {
        const docId = target.id.split('-')[1];
        this.reanalyze(docId);
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
    EventBus.$off('admin:csafAggregators:rowUpdated');
    EventBus.$off('admin:csafAggregators:rowDeleted');
    EventBus.$off('admin:csafProviders:rowUpdated');
    EventBus.$off('admin:csafProviders:rowDeleted');
    EventBus.$off('admin:csafDocuments:rowUpdated');
    EventBus.$off('admin:csafDocuments:rowDeleted');
  },
};
</script>
