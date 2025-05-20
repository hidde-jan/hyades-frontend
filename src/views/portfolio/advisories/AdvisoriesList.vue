<template>
  <div class="componentSearch animated fadeIn" v-permission="'VIEW_PORTFOLIO'">
    <portfolio-widget-row :fetch="true" />

    <bootstrap-table
      ref="table_advisories"
      :columns="columns"
      :data="data"
      :options="options"
      data-click-to-select="true"
      @on-click-row="handleRowClick"
    >
  </div>
</template>

<script>
import PortfolioWidgetRow from '../../dashboard/PortfolioWidgetRow';
import permissionsMixin from '../../../mixins/permissionsMixin';


export default {
  mixins: [permissionsMixin],
  components: {
    PortfolioWidgetRow,
  },
  beforeCreate() {

  },
  beforeMount() {

  },
  watch: {

  },
  methods: {
    handleRowClick(row) {
    this.$router.push({ name: '/advisories/advisoryDetail', params: { id: row.name } });
    },
  },
  data() {
    return {
      columns: [
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
        {
          select: true,
          name: 'Intevation GmbH',
          url: 'https://intevation.de/.well-known/csaf/provider-metadata.json',
          projects: 5,
          matches: 3,
        },
        {
          select: false,
          name: 'CSIA',
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
};
</script>

<style>
.componentSearch .bootstrap-table .fixed-table-toolbar .bs-bars {
  width: 80%;
}
</style>
