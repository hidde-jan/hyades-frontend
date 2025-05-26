<template>
  <div>
    <bootstrap-table
      ref="advisoriesTable"
      :columns="columns"
      :data="data"
      :options="options"
    />
    <b-button size="md" variant="outline-primary" @click="handleSelected">
      <span class="fa fa-plus"></span>
      Add audit details
    </b-button>
    <project-advisories-add-info-modal />
  </div>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import $ from 'jquery';
import xssFilters from 'xss-filters';

import common from '@/shared/common';
import i18n from '@/i18n';
import {
  compareVersions,
  loadUserPreferencesForBootstrapTable,
} from '@/shared/utils';
import bootstrapTableMixin from '@/mixins/bootstrapTableMixin';
import permissionsMixin from '@/mixins/permissionsMixin';
import FindingAudit from './FindingAudit';
import ProjectUploadVexModal from './ProjectUploadVexModal';
import ProjectAdvisoriesAddInfoModal from './ProjectAdvisoriesAddInfoModal.vue';

export default {
  props: {
    uuid: String,
  },
  mixins: [bootstrapTableMixin, permissionsMixin],
  components: {
    cSwitch,
    ProjectUploadVexModal,
    ProjectAdvisoriesAddInfoModal,
  },
  data() {
    return {
      showSuppressedFindings: this.showSuppressedFindings,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      columns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          field: 'name',
          title: this.$t('message.advisory'),
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../advisories/' + encodeURIComponent(row.documentId),
            );
            return `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
          },
        },
        {
          field: 'url',
          title: this.$t('message.analyzer'),
          sortable: true,
        },
        /*{
          field: 'documentId',
          title: this.$t('message.matchpercentage'),
          sortable: true,
        },*/
        {
          field: 'findingsPerDoc',
          title: this.$t('message.affected_components'),
          sortable: true,
        },
      ],
      data: [],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        silentSort: false,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        icons: {
          detailOpen: 'fa-fw fa-angle-right',
          detailClose: 'fa-fw fa-angle-down',
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: true,
        detailViewByClick: false,
        /* detailFormatter: (index, row) => {
          return (
            row &&
            this.vueFormatter({
              i18n,
              propsData: {
                finding: row,
                projectUuid: this.uuid,
              },
              ...FindingAudit,
            })
          );
        },
        onExpandRow: this.vueFormatterInit, */
      },
    };
  },
  mounted() {
    this.refreshTable();
  },
  methods: {
    apiUrl: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/project/${this.uuid}`;
      if (this.showSuppressedFindings === undefined) {
        url += '?suppressed=false';
      } else {
        url += '?suppressed=' + this.showSuppressedFindings;
      }
      return url;
    },
    /*reAnalyze: function (data) {
      let analyzeUrl = `${this.$api.BASE_URL}/${this.$api.URL_FINDING}/project/${this.uuid}/analyze`;
      this.axios.post(analyzeUrl).then((response) => {
        this.$toastr.s(this.$t('message.project_reanalyze_requested'));
        //ignore token from response, don't wait for completion
        this.refreshTable();
      });
    },*/
    refreshTable: function () {
      this.$refs.advisoriesTable.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    handleSelected() {
      console.log('click');
      const selectedRows = this.$refs.advisoriesTable.getSelections();
      if (selectedRows.length > 0) {
        console.log('launching modal');
        this.$bvModal.show('projectAdvisoriesAddInfoModal', {
          props: {
            selectedRows: selectedRows,
          },
        });
      } else {
        alert('Select at least one row');
      }
    },
  },
};
</script>
