<template>
  <iframe
    ref="grader"
    class="mt-2 border border-white"
    src="/grader/ephemeral/index-light.html?embedded"
  ></iframe>
</template>

<script lang="ts">
import { Component, Vue, Prop, Ref, Watch } from 'vue-property-decorator';
import { types } from '../../api_types';

interface CSSStyleDeclaration {
  zoom?: string | number;
}

@Component
export default class EphemeralGrader extends Vue {
  @Ref() grader!: HTMLIFrameElement;
  @Prop() problem!: types.ProblemInfo;
  @Prop({ default: false }) canSubmit!: boolean;
  @Prop({ default: () => [] }) acceptedLanguages!: string[];
  @Prop({ default: 'cpp17-gcc' }) preferredLanguage!: string;

  loaded = false;

  mounted(): void {
    (this.$refs.grader as HTMLIFrameElement).onload = () => {
      const languageSelectElement: HTMLSelectElement = ((this.$refs
        .grader as HTMLIFrameElement)
        .contentWindow as Window).document.getElementById(
        'language',
      ) as HTMLSelectElement;
      if (!this.acceptedLanguages.includes(this.preferredLanguage)) {
        languageSelectElement.value = this.acceptedLanguages[0];
      } else {
        languageSelectElement.value = this.preferredLanguage;
      }
      this.iframeLoaded();
    };
    window.addEventListener('resize', this.handleWindowResize);
  }
  unmounted(): void {
    window.removeEventListener('resize', this.handleWindowResize);
  }
  handleWindowResize(): void {
    const iframe = this.$refs.grader as HTMLIFrameElement;

    (iframe.style as CSSStyleDeclaration).zoom = String(
      1 / window.devicePixelRatio,
    );
  }
  @Watch('problem')
  onProblemChanged() {
    if (!this.loaded) {
      // This will be updated when the component is mounted.
      return;
    }
    this.setSettings();
  }

  setSettings(): void {
    ((this.$refs.grader as HTMLIFrameElement)
      .contentWindow as Window).postMessage(
      {
        method: 'setSettings',
        params: {
          alias: this.problem.alias,
          settings: this.problem.settings,
          languages: this.acceptedLanguages,
          showSubmitButton: this.canSubmit,
        },
      },
      `${window.location.origin}/grader/ephemeral/embedded/`,
    );
  }

  iframeLoaded(): void {
    this.loaded = true;
    this.setSettings();
  }
}
</script>

<style lang="scss" scoped>
iframe {
  width: 100%;
  min-height: 60em;
}
</style>
