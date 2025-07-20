<template>
  <bootstrap-table
    ref="tableAdvisoryFindings"
    :columns="taColumns"
    :data="taData"
    :options="taOptions"
  />
</template>

<script>
import BootstrapToggle from 'vue-bootstrap-toggle';
import permissionsMixin from '@/mixins/permissionsMixin';
import xssFilters from 'xss-filters';

export default {
  props: {
    row: Object,
    projectUuid: String,
  },
  data() {
    return {
      taColumns: [
        {
          title: 'Component',
          field: 'name',
          sortable: true,
          formatter: (value, row, index) => {
            let url = xssFilters.uriInUnQuotedAttr(
              '../../../components/' + row.componentUuid,
            );
            let dependencyGraphUrl = xssFilters.uriInUnQuotedAttr(
              '../../../projects/' +
                this.projectUuid +
                '/dependencyGraph/' +
                row.componentUuid,
            );
            return (
              `<a href="${dependencyGraphUrl}"<i class="fa fa-sitemap" aria-hidden="true" style="float:right; padding-top: 4px; cursor:pointer" data-toggle="tooltip" data-placement="bottom" title="Show in dependency graph"></i></a> ` +
              `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`
            );
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
          formatter: (cell) => {
            return `${(cell * 100).toFixed(0)}%`;
          },
        },
        /*{
          title: 'Vulnerability',
          field: 'vulnerability',
        },*/
      ],
      taData: [],
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
        /*detailView: true,
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
        onExpandRow: this.vueFormatterInit,*/
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
      this.$refs.tableAdvisoryFindings.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
  },
  mounted() {
    this.refreshTable();
  },
  components: {
    BootstrapToggle,
  },
};
</script>
