<template>
  <div class="animated fadeIn">
    <b-card :no-body="true">
      <b-card-body class="p-3 clearfix">
        <div class="h5 mb-0 mt-2">{{ advisory.name }}</div>
      </b-card-body>
    </b-card>

    <b-tabs class="body-bg-color">
      <b-tab title="Overview">
        <h1>{{ advisory.name }}</h1>
        <h2>{{ advisory.trackingID }}</h2>
        <h3>{{ advisory.trackingVersion }}</h3>
        <h4>{{ advisory.lastFetched }}</h4>
        <p>{{ advisory.url }}</p>
        <h2>Matches: {{nMatches}}</h2>
        <h2>With status set: {{nStatus}}</h2>
      </b-tab>
      <b-tab title="Affected Projects">
        <bootstrap-table
          ref="table_projects"
          :columns="columns"
          :data="affectedProjects"
          :options="options"
        />
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import xssFilters from 'xss-filters';
import EventBus from '../../../shared/eventbus';

export default {
  data() {
    return {
      nMatches: 0,
      nStatus: 0,
      advisoryId: null,
      advisory: {},
      columns: [
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../projects/' + row.uuid + '/advisories',
            );
            return `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
          },
        },
        {
          title: 'Version',
          field: 'version',
          class: 'tight',
          sortable: true,
          width: '350px',
        },
        {
          title: 'Description',
          field: 'desc',
          align: 'center',
          sortable: true,
        },
      ],
      affectedProjects: [],
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
  methods: {
    apiUrl: function () {
      //TODO: find correct url
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/${this.advisoryId}`;
      return url;
    },
    initializeData: function () {
      this.advisoryId = this.$route.params.advisoryId;
      //this.vulnId = decodeURIComponent(this.$route.params.advisoryId);
    },
    loadData: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        this.advisory = response.data.entity;
        this.nMatches = response.data.findingsTotal;
        this.nStatus = response.data.findingsMarked;
        this.affectedProjects = response.data.affectedProjects;
        EventBus.$emit('addCrumb', this.advisory.name);
        this.$title = this.advisory.name;
      });
    },
    routeTo(path) {
      if (path) {
        if (
          !this.$route.fullPath.toLowerCase.includes('/' + path.toLowerCase())
        ) {
        // TODO enable tab routing
        }
      }
    },
  },
  watch: {
    '$route.params.advisoryId'(newValue) {
      EventBus.$emit('crumble');
      this.initializeData();
      this.loadData();
    },
    $route() {
      //this.getTabFromRoute().activate();
    },
  },
  beforeMount() {
    this.advisoryId = this.$route.params.advisoryId;
    this.initializeData();
  },
  mounted() {
    /*try {
      if (this.$route.params.componentUuids) {
        this.$refs.dependencygraph.active = true;
      } else {
        this.getTabFromRoute().active = true;
      }
    } catch (e) {
      this.$toastr.e(this.$t('condition.forbidden'));
      this.$router.replace({ path: '/projects/' + this.uuid });

      // TODO activate appropriate tab this.$refs.overview.active = true;
    }*/
    //this.refreshTable();
    this.loadData();
    this.getStats();
  },
  refreshTable: function () {
    //TODO uncomment when api url available
    /* this.$refs.table_projects.refresh({
         url: this.apiUrl(),
         pageNumber: 1,
         silent: true,
       });*/
  },
  destroyed() {
    EventBus.$emit('crumble');
  },
};
</script>

<style scoped></style>
