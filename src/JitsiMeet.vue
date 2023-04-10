<template>
  <div ref="jitsiContainer" style="height: 100%; width: 100%;"></div>
</template>

<script>
export default {
  props: {
    domain: {
      type: String,
      default: 'meet.jit.si'
    },
    options: {
      type: Object,
      default: () => ({}),
    },
  },
  data () {
    return {
      jitsiApi: null,
    };
  },
  mounted () {
    this.loadScript(`https://${this.domain}/external_api.js`, () => {
      if (!window.JitsiMeetExternalAPI) throw new Error('Jitsi Meet External API not loaded');
      this.embedJitsiWidget();
    });
  },
  beforeDestroy () {
    this.removeJitsiWidget();
  },
  methods: {
    loadScript (src, cb) {
      const scriptEl = document.createElement('script');
      scriptEl.src = src;
      scriptEl.async = 1;
      document.querySelector('head').appendChild(scriptEl);
      scriptEl.addEventListener('load', cb);
    },
    embedJitsiWidget () {
      const options = {
        ...this.options,
        parentNode: this.$refs.jitsiContainer,
      };
      this.jitsiApi = new window.JitsiMeetExternalAPI(this.domain, options);
    },
    executeCommand (command, ...value) {
      this.jitsiApi.executeCommand(command, ...value);
    },
    addEventListener (event, fn) {
      this.jitsiApi.on(event, fn);
    },
    // Misc
    removeJitsiWidget () {
      if (this.jitsiApi) this.jitsiApi.dispose();
    },
  }
};
</script>
