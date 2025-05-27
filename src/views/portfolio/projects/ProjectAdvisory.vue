<template>
  <bootstrap-table
    ref="tableAdvisoryFindings"
    :columns="taColumns"
    :data="taData"
    :options="taOptions"
  />
</template>

<script>
import common from '@/shared/common';
import BootstrapToggle from 'vue-bootstrap-toggle';
import permissionsMixin from '@/mixins/permissionsMixin';
import xssFilters from 'xss-filters';
import FindingAudit from './FindingAudit';
import i18n from '@/i18n';

export default {
  props: {
    row: Object,
  },
  data() {
    return {
      taColumns: [
        {
          title: 'Component',
          field: 'component',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../advisories/' + encodeURIComponent(row.documentId),
            );
            return `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
          },
        },
        {
          title: 'Version',
          field: 'version',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Group',
          field: 'group',
          sortable: true,
        },
        {
          title: 'Confidence',
          field: 'confidence',
          sortable: true,
        },
        {
          title: 'Vulnerability',
          field: 'vulnerability',
        },
      ],
      taData: [
        {
          component: 'Sample component',
          version: 'example.com',
          group: '42',
          confidence: '80%',
        },
      ],
      taOptions: {
        search: false,
        showColumns: true,
        showRefresh: false,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'name',
        sortOrder: 'asc',
        icons: {
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: true,
        detailViewByClick: false,
        detailFormatter: (index, row) => {
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
        onExpandRow: this.vueFormatterInit,
      },
    };
  },
  watch: {},
  mixins: [permissionsMixin],
  methods: {
    apiUrl: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/project/${this.row.projectId}/advisory/${this.row.documentId}`;
      return url;
    },
    refreshTable: function () {
      console.log("attempting retrieval of "+this.apiUrl());
      this.$refs.tableAdvisoryFindings.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
  },
  beforeMount() {},
  mounted() {
    console.log("mounting with "+this.row.projectId+" advisory "+this.row.documentId)
    this.refreshTable();
  },
  components: {
    BootstrapToggle,
  },
};
</script>
