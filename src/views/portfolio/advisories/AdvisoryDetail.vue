<template>
  <div class="animated fadeIn">
    <b-card :no-body="true">
      <b-card-body class="p-3 clearfix">
        <div class="h5 mb-0 mt-2">{{ vulnLabel }}</div>
      </b-card-body>
    </b-card>

    <b-tabs class="body-bg-color">
      <b-tab title="Overview">
        <p>{{ overviewContent }}</p>
      </b-tab>
      <b-tab title="Affected Projects">
        <bootstrap-table
          ref="table_projects"
          :columns="columns"
          :data="data"
          :options="options"
        >
        </bootstrap-table>
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import xssFilters from 'xss-filters';

export default {
  props: ['id'],
  mounted() {
    console.log(this.id);
    this.refreshTable();
  },
  methods: {
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
      vulnLabel: this.id,
      overviewContent: 'Some content',
      columns: [
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter: (value, row) => {
            const url = this.$router.resolve({
              name: 'Project Vulnerability Lookup',
              params: { uuid: row.uuid, vulnerability: this.vulnerability },
            }).href;

            let html = `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
            if (row.dependencyGraphAvailable) {
              const dependencyGraphUrl = this.$router.resolve({
                name: 'Dependency Graph Component Lookup',
                params: {
                  uuid: row.uuid,
                  componentUuids: row.affectedComponentUuids.join('|'),
                },
              }).href;
              html =
                `<a href="${dependencyGraphUrl}"><i class="fa fa-sitemap" aria-hidden="true" style="float:right; padding-top: 4px; cursor:pointer" data-toggle="tooltip" data-placement="bottom" title="Show in dependency graph"></i></a> ` +
                html;
            }

            return html;
          },
        },
        {
          title: 'Version',
          field: 'tag',
          class: 'tight',
          sortable: true,
          width: '350px',
        },
        {
          title: 'Active',
          field: 'active',
          formatter(value, row, index) {
            return value === true ? '<i class="fa fa-check-square-o" />' : '';
          },
          align: 'center',
          sortable: true,
        },
      ],
      data: [
        //TODO: delete when api is working
        {
          name: 'Project A',
          tag: '1',
          active: true,
        },
        {
          name: 'Project B',
          tag: '1.3',
          active: false,
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
      },
    };
  },
};
</script>

<style scoped></style>
