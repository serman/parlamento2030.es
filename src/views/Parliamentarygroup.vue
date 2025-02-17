<template>
  <div
    v-if="parliamentarygroup"
    id="group"
    class="o-container o-section u-margin-bottom-10"
  >
    <tipi-header v-if="parliamentarygroup" :title="parliamentarygroup.name" />

    <div class="o-container o-zection u-margin-top-8 u-margin-bottom-4">
      <SdgBarchartFootprint
        v-if="footprintByTopics.length > 0"
        :footprint="footprintByTopics"
        :styles="topicsStyles"
        :linkToSearchField="'author'"
        :linkToSearchValue="parliamentarygroup.name"
      />
      <footprint-info />
    </div>

    <div
      v-if="latestInitiatives && latestInitiatives.length"
      class="o-container o-section u-margin-bottom-4"
    >
      <h4 class="u-margin-bottom-4">Últimas iniciativas</h4>
      <tipi-results
        layout="tiny"
        :initiatives="latestInitiatives"
        :topicsStyles="topicsStyles"
      />
    </div>
    <div class="o-container o-section" v-else>
      <tipi-message type="info" icon>
        Sin actividad parlamentaria relacionada con la Agenda 2030 en esta
        legislatura
      </tipi-message>
    </div>

    <alert-block
      :text="'No te pierdas nada de la actividad del '"
      :value="parliamentarygroup.name"
      :searchparams="{ author: parliamentarygroup.name }"
      v-if="use_alerts"
    />

    <div class="o-container o-section">
      <h4 class="u-margin-bottom-4">Diputados/as</h4>
      <div class="o-grid">
        <div class="o-grid__col u-12 u-4@sm">
          <tipi-text
            meta=""
            :value="this.dividedDeputies[0]"
            type="deputy"
            :source="this.store.allDeputies"
            hideGroup
          />
        </div>
        <div class="o-grid__col u-12 u-4@sm">
          <tipi-text
            meta=""
            :value="this.dividedDeputies[1]"
            type="deputy"
            :source="this.store.allDeputies"
            hideGroup
          />
        </div>
        <div class="o-grid__col u-12 u-4@sm">
          <tipi-text
            meta=""
            :value="this.dividedDeputies[2]"
            type="deputy"
            :source="this.store.allDeputies"
            hideGroup
          />
        </div>
      </div>
    </div>
  </div>
  <div v-else class="o-container o-section u-margin-bottom-10">
    <tipi-loader title="Cargando datos" subtitle="Puede llevar unos segundos" />
  </div>
</template>

<script>
import {
  TipiHeader,
  TipiMessage,
  TipiResults,
  TipiText,
  TipiLoader,
} from '@politicalwatch/tipi-uikit';
import SdgBarchartFootprint from '@/components/sdg-barchart-footprint.vue';
import FootprintInfo from '@/components/FootprintInfo.vue';
import AlertBlock from '@/components/alert-block.vue';
import api from '@/api';
import config from '@/config';
import { useParliamentStore } from '@/stores/parliament';

export default {
  name: 'parliamentarygroup',
  components: {
    TipiHeader,
    TipiMessage,
    TipiResults,
    TipiText,
    TipiLoader,
    SdgBarchartFootprint,
    FootprintInfo,
    AlertBlock,
  },
  setup() {
    const store = useParliamentStore();
    return { store };
  },
  data: function () {
    return {
      parliamentarygroup: null,
      latestInitiatives: null,
      topicsRanking: null,
      use_alerts: config.USE_ALERTS,
      topicsStyles: config.STYLES.topics,
    };
  },
  computed: {
    deputies: function () {
      if (this.parliamentarygroup) {
        return this.store
          .getDeputiesByParliamentaryGroup(this.parliamentarygroup.shortname)
          .filter((deputy) => deputy.active)
          .map((deputy) => deputy.name);
      }
      return [];
    },
    dividedDeputies: function () {
      let results = [];
      let divided = this.deputies;

      for (let i = 3; i > 0; i--) {
        results.push(divided.splice(0, Math.ceil(divided.length / i)));
      }

      return results;
    },
    footprintByTopics: function () {
      if (this.parliamentarygroup) {
        return this.parliamentarygroup.footprint_by_topics.filter((item) =>
          this.store.allTopics.some((topic) => topic.name === item.name)
        );
      }
      return [];
    },
  },
  methods: {
    getParliamentaryGroup: function () {
      api
        .getGroup(this.$route.params.id)
        .then((response) => {
          this.parliamentarygroup = response;
          this.getTopicsRanking();
          this.getLatestInitiatives();
        })
        .catch((error) => {
          this.errors = error;
          this.$router.push({ name: 'Page404', params: { 0: '404' } });
        });
    },
    getLatestInitiatives: function () {
      api
        .getInitiatives({ author: this.parliamentarygroup.name, per_page: 12 })
        .then((response) => {
          if (response.initiatives)
            this.latestInitiatives = response.initiatives;
        })
        .catch((error) => (this.errors = error));
    },
    getTopicsRanking: function () {
      api
        .getTopicsByParliamentaryGroupRanking(this.parliamentarygroup.name)
        .then((response) => {
          this.topicsRanking = response;
        })
        .catch((error) => (this.errors = error));
    },
  },
  metaInfo() {
    const title = this.parliamentarygroup?.name
      ? `${this.parliamentarygroup.name} - Qué hacen los diputados`
      : '- Qué hacen los diputados';

    return {
      title,
      meta: [
        {
          property: 'og:title',
          content: title,
          vmid: 'og:title',
        },
      ],
    };
  },
  created: function () {
    this.getParliamentaryGroup();
  },
};
</script>
