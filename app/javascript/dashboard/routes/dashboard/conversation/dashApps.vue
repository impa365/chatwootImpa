<script>
// import alertMixin from 'shared/mixins/alertMixin';
import LoadingState from 'dashboard/components/widgets/LoadingState.vue';
import { mapGetters } from 'vuex';

export default {
  components: {
    LoadingState,
  },
  props: {
    dashAppId: {
      type: String,
    },
  },
  data() {
    return {
      hasOpenedAtleastOnce: false,
      iframeLoading: true,
      mostrarIframe: false,
      iframeUrl: '',
    };
  },
  computed: {
    ...mapGetters({
      dashboardApps: 'dashboardApps/getRecords',
    }),
  },
  watch: {
    dashAppId: 'getUrl',
    'dashboardApps.id'() {
      this.$store.dispatch('dashboardApps/get');
    },
  },
  mounted() {
    this.$store.dispatch('dashboardApps/get');

    /* window.onmessage = e => {
      if (
        typeof e.data !== 'string' ||
        e.data !== 'chatwoot-dashboard-app:fetch-info'
      ) {
        return;
      }
      this.onIframeLoad(0);
    }; */

    this.getUrl();
  },
  methods: {
    async getUrl() {
      let dashApp = this.dashboardApps.filter(
        d => d.id == parseInt(this.dashAppId)
      );
      this.iframeUrl = dashApp[0].content[0].url;
      this.mostrarIframe = true;
      this.iframeLoading = false;
    },
    onIframeLoad() {
      console.log('Iframe Load: ' + this.iframeUrl);
    },
  },
};
</script>

<template>
  <div class="dashboard-app--container">
    <LoadingState
      v-if="iframeLoading"
      :message="$t('Carregando Integração...')"
      class="dashboard-app_loading-container"
    />
    <iframe
      v-if="mostrarIframe"
      :src="iframeUrl"
      @load="() => onIframeLoad()"
    />
  </div>
</template>

<style scoped>
iframe {
  width: 100%;
  height: 100%;
  border: 0;
}

.dashboard-app--container,
.dashboard-app--list,
.dashboard-app--list iframe {
  height: 100%;
  width: 100%;
}

.dashboard-app--list iframe {
  border: 0;
}
.dashboard-app_loading-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
}
</style>
