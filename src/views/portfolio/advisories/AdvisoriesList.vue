<template>
  <div class="componentSearch animated fadeIn" v-permission="'VIEW_PORTFOLIO'">
    <portfolio-widget-row :fetch="true" />

    <bootstrap-table
      ref="table_advisories"
      :columns="columns"
      :data="data"
      :options="options"
      data-click-to-select="true"
    >
      <template v-slot:cell(name)="data">
        <a @click.prevent="handleRowClick(data.row)">{{ data.value }}</a>
      </template>
    </bootstrap-table>
  </div>
</template>

<script>
import PortfolioWidgetRow from '../../dashboard/PortfolioWidgetRow';
import permissionsMixin from '../../../mixins/permissionsMixin';
import xssFilters from 'xss-filters';

export default {
  mixins: [permissionsMixin],
  components: {
    PortfolioWidgetRow,
  },
  beforeCreate() {},
  beforeMount() {},
  watch: {},
  methods: {
    handleRowClick(row) {
      this.$router.push({ name: '/advisory', params: { id: row.name } });
    },
    apiUrl: function () {
      //TODO: find correct url
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/project/${this.uuid}`;
      if (this.showSuppressedFindings === undefined) {
        url += '?suppressed=false';
      } else {
        url += '?suppressed=' + this.showSuppressedFindings;
      }
      return url;
    },
    refreshTable: function () {
      //TODO uncomment when api url available
     /* this.$refs.table_advisories.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });*/
    },
  },
  data() {
    return {
      columns: [
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr('../advisory/' + row.name);
            return `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
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
          title: 'Projects',
          field: 'projects',
          sortable: true,
        },
        {
          title: 'Matches',
          field: 'matches',
          sortable: true,
        },
      ],
      data: [
        //TODO: delete when api is working
        {
          select: true,
          name: 'Intevation GmbH',
          url: 'https://intevation.de/.well-known/csaf/provider-metadata.json',
          projects: 5,
          matches: 3,
        },
        {
          select: false,
          name: 'CISA',
          url: 'https://www.cisa.gov/sites/default/files/csaf/provider-metadata.jsona',
          projects: 2,
          matches: 1,
        },
        {
          select: true,
          name: 'Nazomine Networks',
          url: 'https://csaf.data.security.nozominetworks.com/provider-metadata.json',
          projects: 8,
          matches: 4,
          seen: false,
          new: 'New',
        },
      ],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
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
        onClickRow: this.handleRowClick,
      },
    };
  },
  mounted() {
    this.refreshTable();
  },
};
</script>

<style>
.componentSearch .bootstrap-table .fixed-table-toolbar .bs-bars {
  width: 80%;
}
</style>
