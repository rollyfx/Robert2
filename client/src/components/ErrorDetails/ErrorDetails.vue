<template>
  <div class="ErrorDetails">
    <div class="ErrorDetails__header">
      <h2 class="ErrorDetails__title">
        {{ $t('errors.details-title') }}
      </h2>
      <button class="ErrorDetails__btn-close" @click="$emit('close')">
        <i class="fas fa-times" />
      </button>
    </div>
    <div class="ErrorDetails__main">
      <p>
        {{ $t('errors.details-intro1') }}
        {{ $t('errors.details-intro2') }}
        <a href="https://forum.robertmanager.org" target="_blank">
          {{ $t('errors.details-intro-forum') }}
        </a>
        {{ $t('errors.details-intro3') }}
        <a href="https://gitlab.com/robertmanager/Robert2-API/-/issues" target="_blank">GitLab</a>.
      </p>
      <p v-if="!data.requested" class="ErrorDetails__no-details-info">
        <i class="fas fa-info-circle" />
        {{ $t('errors.details-intro-not-detailed') }}
      </p>
      <hr />
      <div class="ErrorDetails__content" ref="errorContent">
        <div v-if="data.requested">
          <h3 class="ErrorDetails__subtitle">
            #### {{ $t('errors.details-request') }}
          </h3>
          <p class="ErrorDetails__request">
            `{{ data.requested }}`
          </p>
        </div>
        <h3 class="ErrorDetails__subtitle">
          #### {{ $t('errors.details-message') }}
          (<span class="ErrorDetails__code">{{ data.code }}</span>) :
        </h3>
        <p class="ErrorDetails__message">
          `{{ data.message }}`
        </p>
        <div v-if="file.length > 0">
          <h3 class="ErrorDetails__subtitle">
            #### {{ $t('errors.details-file') }}
          </h3>
          <p class="ErrorDetails__file">
            `{{ file }}`
          </p>
        </div>
        <div v-if="trace.length > 0">
          <h3 class="ErrorDetails__subtitle">
            #### {{ $t('errors.details-stacktrace') }}
          </h3>
          <div class="ErrorDetails__trace">
            >! Details<br />
            ```log<br />
            <div v-for="(traceItem, index) in trace" :key="index" class="ErrorDetails__traceItem">
              <span class="ErrorDetails__traceItem__index">- {{ index }}:&nbsp;</span>
              <span class="ErrorDetails__traceItem__info">
                <span v-if="traceItem.class">{{ traceItem.class }}::</span>{{ traceItem.function }}
                <br />
              </span>
              <span v-if="traceItem.file" class="ErrorDetails__traceItem__info">
                File: {{ traceItem.file }}, line {{ traceItem.line }}
              </span>
            </div>
            ```
          </div>
        </div>
      </div>
      <hr />
      <div class="ErrorDetails__footer">
        <p v-if="isCopied" class="ErrorDetails__is-copied">
          <i class="fas fa-check" />
          {{ $t('copied-in-clipboard') }}
        </p>
        <button @click="copyErrorContent" class="info">
          <i class="fas fa-clipboard" />
          {{ $t('copy-to-clipboard') }}
        </button>
        <button @click="$emit('close')">
          <i class="fas fa-times" />
          {{ $t('close') }}
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
  @import '../../themes/default/index';
  @import './ErrorDetails';
</style>

<script>
const cleanFilePath = (path) => {
  if (!path) {
    return '';
  }

  const filePath = path.split('/src/');
  return filePath.length > 0 ? `src/${filePath[1]}` : path;
};

export default {
  name: 'ErrorDetails',
  props: ['data'],
  data() {
    return {
      isCopied: false,
    };
  },
  computed: {
    file() {
      return cleanFilePath(this.data.file);
    },

    trace() {
      if (!this.data.stackTrace) {
        return [];
      }

      return this.data.stackTrace.map((traceItem) => {
        const { file } = traceItem;
        return { ...traceItem, file: cleanFilePath(file) };
      });
    },
  },
  methods: {
    copyErrorContent() {
      const range = document.createRange();
      range.selectNode(this.$refs.errorContent);
      window.getSelection().removeAllRanges();
      window.getSelection().addRange(range);
      document.execCommand('copy');

      this.isCopied = true;
      window.getSelection().removeAllRanges();
    },
  },
};
</script>
