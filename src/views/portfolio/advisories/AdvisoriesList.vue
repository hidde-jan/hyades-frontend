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
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}`;
      return url;
    },
    refreshTable: function () {
      this.$refs.table_advisories.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
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
            let url = xssFilters.uriInUnQuotedAttr(
              '../advisories/' + encodeURIComponent(row.documentId),
            );
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
          field: 'affectedProjects',
          sortable: true,
        },
        {
          title: 'Matches',
          field: 'affectedComponents',
          sortable: true,
        },
      ],
      data: [],
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
      },
    };
  },
  mounted() {
    this.refreshTable();
    /*this.$refs.table_advisories.$el.addEventListener('click', (event) => {
      const target = event.target;
    });*/
  },
};
</script>
