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
      //TODO uncomment when api url available
      this.$refs.table_advisories.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
    reanalyze(adId) {
      console.log(`Reanalyzing advisory with ID: ${adId}`);
      //TODO: Add api call
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
        /*{
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="reanalyze-${row.name}"> <span class="fa fa-refresh"></span> Reanalyze </button>`;
          },
        },*/
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
        //onClickRow: this.handleRowClick,
      },
    };
  },
  mounted() {
    this.refreshTable();
    /*this.$refs.table_advisories.$el.addEventListener('click', (event) => {
      const target = event.target;
      if (target.matches('[id^="reanalyze-"]')) {
        const adId = target.id.split('-')[1];
        this.reanalyze(adId);
      }
    });*/
  },
};
</script>
