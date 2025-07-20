<template>
  <div class="animated fadeIn">
    <b-card :no-body="true">
      <b-card-body class="p-3 clearfix">
        <div class="h5 mb-0 mt-2">{{ advisory.name }}</div>
      </b-card-body>
    </b-card>

    <b-tabs class="body-bg-color">
      <b-tab title="Overview">
        <b-card>
          <table>
            <tr>
              <th>Name</th>
              <td>
                {{ advisory.name }}
              </td>
            </tr>
            <tr>
              <th>Tracking ID</th>
              <td>{{ advisory.trackingID }}</td>
            </tr>
            <tr>
              <th>Version</th>
              <td>{{ advisory.trackingVersion }}</td>
            </tr>
            <tr>
              <th>Last Fetched</th>
              <td>{{ formatDate(advisory.lastFetched) }}</td>
            </tr>
            <tr>
              <th>URL</th>
              <td>
                <a :href="advisory.url" target="_blank">{{ advisory.url }}</a>
              </td>
            </tr>
          </table>
        </b-card>
        <b-card title="Publisher">
          <table>
            <tr>
              <th>Name</th>
              <td>
                {{ doc.document.publisher.name }}
              </td>
            </tr>
            <tr>
              <th>Namespace</th>
              <td>
                {{ doc.document.publisher.namespace }}
              </td>
            </tr>
            <tr>
              <th>Category</th>
              <td>
                {{ doc.document.publisher.category }}
              </td>
            </tr>
          </table>
        </b-card>
        <b-card
          :title="value.category"
          v-for="(value, key) in doc.document.notes"
          :key="key"
        >
          <b-card-text>
            {{ value.text }}
          </b-card-text>
        </b-card>
        <pre>{{ JSON.stringify(doc, null, 2) }}</pre>
      </b-tab>
      <b-tab title="Affected Projects">
        <bootstrap-table
          ref="table_projects"
          :columns="projectsColumns"
          :data="affectedProjects"
          :options="options"
        />
      </b-tab>
      <b-tab title="Vulnerabilities">
        <bootstrap-table
          ref="table_vulnerabilities"
          :columns="vulnerabilitiesColumns"
          :data="vulnerabilities"
          :options="options"
        />
      </b-tab>
      <b-tab title="Document History">
        <b-card
          :title="value.date"
          v-for="(value, key) in doc.document.tracking.revision_history"
          :key="key"
        >
          <b-card-text>
            {{ value.summary }}
          </b-card-text>
        </b-card>
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
      doc: {},
      projectsColumns: [
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
      vulnerabilitiesColumns: [
        {
          title: 'ID',
          field: 'vulnId',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../vulnerabilities/' + row.source + '/' + row.vulnId,
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
      vulnerabilities: [],
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
    formatDate(value) {
      const date = new Date(value * 1000);
      return date.toLocaleString();
    },
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
        this.doc = JSON.parse(response.data.entity.content);
        this.affectedProjects = response.data.affectedProjects;
        this.vulnerabilities = response.data.vulnerabilities;
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
