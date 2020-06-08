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
      type: String,
      default: () => ({}),
    },
  },
  mounted () {
    this.loadScript('https://meet.jit.si/external_api.js', () => {
      if (!window.JitsiMeetExternalAPI) throw new Error('Jitsi Meet API not loaded');
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
