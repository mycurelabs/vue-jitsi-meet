<template>
  <div ref="jitsi" style="height: 100%; width: 100%;"></div>
</template>

<script>
export default {
  props: {
    roomName: {
      type: String,
      default: null
    },
    subject: {
      type: String,
      default: null
    },
    width: {
      type: [Number, String],
      default: '100%'
    },
    height: {
      type: [Number, String],
      default: '100%'
    },
    jitsiMeetDomain: {
      type: String,
      default: 'meet.jit.si'
    },
    participant: {
      type: Object,
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
      // build options
      const options = {
        roomName: this.roomName,
        width: this.width,
        height: this.height,
        parentNode: this.$refs.jitsi,
        noSSL: false,
        userInfo: {
          email: this.participant.email
        },
        configOverwrite: {
          enableNoisyMicDetection: false
        },
        interfaceConfigOverwrite: {
          SHOW_JITSI_WATERMARK: false,
          SHOW_WATERMARK_FOR_GUESTS: false,
          SHOW_CHROME_EXTENSION_BANNER: false
        },
        onload: this.onIFrameLoad
      };
      if (this.participant.fullName) {
        options.userInfo.displayName = this.participant.fullName;
      }
      // attach iframe
      this.jitsiApi = new window.JitsiMeetExternalAPI(this.jitsiMeetDomain, options);
      // add event listeners
      this.jitsiApi.on('audioMuteStatusChanged', this.audioMuteStatusChanged);
      this.jitsiApi.on('incomingMessage', this.incomingMessage);
      this.jitsiApi.on('outgoingMessage', this.outgoingMessage);
      this.jitsiApi.on('participantJoined', this.participantJoined);
      this.jitsiApi.on('participantKickedOut', this.participantKickedOut);
      this.jitsiApi.on('participantLeft', this.participantLeft);
      this.jitsiApi.on('readyToClose', this.readyToClose);
      this.jitsiApi.on('suspendDetected', this.suspendDetected);
      // run commands
      if (this.subject) this.jitsiApi.executeCommand('subject', this.subject);
    },
    removeJitsiWidget () {
      if (this.jitsiApi) this.jitsiApi.dispose();
    },
    // Events
    onIFrameLoad () {
      this.$emit('onIFrameLoad');
    },
    audioMuteStatusChanged (e) {
      this.$emit('audioMuteStatusChanged', e);
    },
    incomingMessage (e) {
      this.$emit('incomingMessage', e);
    },
    outgoingMessage (e) {
      this.$emit('outgoingMessage', e);
    },
    participantJoined (e) {
      this.$emit('participantJoined', e);
    },
    participantKickedOut (e) {
      this.$emit('participantKickedOut', e);
    },
    participantLeft (e) {
      this.$emit('participantLeft', e);
    },
    readyToClose (e) {
      this.$emit('readyToClose', e);
    },
    suspendDetected (e) {
      this.$emit('suspendDetected', e);
    },
  }
};
</script>
