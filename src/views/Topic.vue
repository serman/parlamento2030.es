<template>
  <div v-if="loaded">
    <tipi-topic-card :topic="topic" :topicsStyles="styles" />

    <div id="topic" class="o-container o-section">
      <div class="o-grid">
        <div class="o-grid__col u-12 u-4@sm" v-if="deputies">
          <tipi-text
            meta="Diputadas/os más activas/os"
            :value="deputies"
            type="deputy"
            :source="this.store.allDeputies"
          />
        </div>
        <div class="o-grid__col u-12 u-4@sm" v-if="parliamentarygroups">
          <tipi-text
            meta="Grupos más activos"
            :value="parliamentarygroups"
            type="parliamentarygroup"
            :source="parliamentarygroups"
          />
        </div>
        <div class="o-grid__col u-12 u-4@sm" v-if="places">
          <tipi-text meta="Dónde se trata más" :value="places" />
        </div>
      </div>
      <div class="u-border-top u-padding-top-4" v-if="latestInitiatives">
        <h4 class="u-margin-bottom-4" v-if="latestInitiatives">
          Últimas iniciativas
        </h4>
        <tipi-results
          layout="tiny"
          :initiatives="latestInitiatives"
          :topicsStyles="styles"
        />
      </div>
    </div>
    <alert-block
      :text="'No te pierdas nada de la actividad parlamentaria relacionada con el '"
      :value="topic.name"
      :searchparams="{ topic: topic.name }"
      v-if="use_alerts"
    />
    <div id="topic" class="o-container o-section">
      <div class="o-grid">
        <div
          class="o-grid__col u-12 u-12@sm"
          v-if="this.styles[topic.name].orgs_logos.length != 0"
        >
          <h4 class="u-margin-bottom-4">Entidades colaboradoras</h4>
        </div>
        <div class="o-grid__col u-12 u-12@sm u-margin-bottom-4">
          <img
            v-for="logo in this.styles[topic.name].orgs_logos"
            :key="logo"
            class="u-padding-right-4"
            :src="'/img/collaborators/' + logo"
            style="height: 100px"
          />
        </div>
      </div>
    </div>
    <div
      class="o-section o-section--double"
      v-if="latestInitiatives"
      :style="`background-color: ${styles[topic.name].color}`"
    >
      <div class="o-container">
        <p class="u-text-subtitle u-margin-0 u-color-white">
          MÁS INICIATIVAS SOBRE
        </p>
        <h4 class="u-text-th2 u-color-white">{{ topic.name.toUpperCase() }}</h4>
        <router-link
          class="c-button c-button--outline"
          :to="{ name: 'results', params: { data: 'topic=' + topic.name } }"
        >
          Explorar
        </router-link>
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
  TipiResults,
  TipiTopicCard,
  TipiText,
  TipiLoader,
} from '@politicalwatch/tipi-uikit';
import AlertBlock from '@/components/alert-block.vue';
import api from '@/api';
import config from '@/config';
import { useParliamentStore } from '@/stores/parliament';

export default {
  name: 'topic',
  components: {
    TipiHeader,
    TipiResults,
    TipiTopicCard,
    TipiText,
    TipiLoader,
    AlertBlock,
  },
  setup() {
    const store = useParliamentStore();
    return { store };
  },
  data: function () {
    return {
      topic: {},
      deputies: null,
      places: null,
      parliamentarygroups: null,
      latestInitiatives: null,
      styles: config.STYLES.topics,
      use_alerts: config.USE_ALERTS,
      loaded: false,
    };
  },
  methods: {
    getTopic: function () {
      api
        .getTopic(this.$route.params.id)
        .then((response) => {
          this.topic = response;
          this.getLatestInitiatives(this.topic.name);
          this.getParliamentarygroupsRanking(this.topic.name);
          this.getDeputiesRanking(this.topic.name);
          this.getPlacesRanking(this.topic.name);
        })
        .catch((error) => {
          this.errors = error;
          this.$router.push({ name: 'Page404', params: { 0: '404' } });
        });
    },
    getDeputiesRanking: function (topic) {
      api
        .getDeputiesRanking(topic, null, 3)
        .then((response) => {
          this.deputies = response;
          this.deputies.forEach((deputy, index) => {
            let foundDeputy = this.store.allDeputies.find(
              (allD) => allD.name === deputy._id
            );
            this.deputies[index] = this.deputies[index]._id;
          });
        })
        .catch((error) => (this.errors = error));
    },
    getPlacesRanking: function (topic) {
      api
        .getPlacesRanking(topic, null, 3)
        .then((response) => {
          this.places = response.map((place) => `${place._id}`);
        })
        .catch((error) => (this.errors = error));
      this.loaded = true;
    },
    getParliamentarygroupsRanking: function (topic) {
      api
        .getParliamentarygroupsRanking(topic)
        .then((response) => {
          this.parliamentarygroups = response;
          this.parliamentarygroups.forEach((group, index) => {
            let foundGroup = this.store.allParliamentaryGroups.find(
              (allPG) => allPG.name === group._id
            );
            this.parliamentarygroups[index].name =
              this.parliamentarygroups[index]._id;
            this.parliamentarygroups[index].id = foundGroup.id;
          });
        })
        .catch((error) => (this.errors = error));
    },
    getLatestInitiatives: function (topic) {
      api
        .getInitiatives({ topic: topic, per_page: 12 })
        .then((response) => {
          if (response.initiatives)
            this.latestInitiatives = response.initiatives;
        })
        .catch((error) => (this.errors = error));
    },
  },
  metaInfo() {
    const title = this.topic?.name
      ? `${this.topic.name} - Parlamento2030`
      : '- Parlamento2030';

    const description = this.topic?.description
      ? this.topic.description
      : 'Parlamento2030 es una innovadora herramienta que rastrea, reúne y ofrece la información sobre la actividad del Congreso de los Diputados español relacionada con los Objetivos de Desarrollo Sostenible. Diseñada para superar los retos que plantea la naturaleza transversal de la Agenda 2030, Parlamento 2030 clasifica la información relacionada con los ODS gracias a un avanzado sistema automático de etiquetado masivo. Esta innovadora tecnología permite a los usuarios navegar por la actividad parlamentaria relacionada con los ODS a través de un buscador online, abierto y gratuito. La información ofrecida es esencial de cara a la monitorización y la rendición de cuentas de la implementación de la Agenda 2030 a nivel nacional.';

    return {
      title,
      meta: [
        {
          property: 'og:title',
          content: title,
          vmid: 'og:title',
        },
        {
          property: 'twitter:title',
          content: title,
          vmid: 'twitter:title',
        },
        {
          vmid: 'description',
          name: 'description',
          content: description,
        },
        {
          property: 'og:description',
          content: description,
          vmid: 'og:description',
        },
      ],
    };
  },
  created: function () {
    this.getTopic();
  },
};
</script>
