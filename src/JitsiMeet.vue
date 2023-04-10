<template>
  <div ref="jitsiContainer" style="height: 100%; width: 100%;"></div>
  <!-- 
    NOTE: This is a work in progress for the low level implementation
   -->
</template>

<script>
export default {
  props: {
    domain: {
      type: String,
      default: 'meet.jit.si'
    },
    libraryUrl: {
      type: String,
      default: 'https://meet.jit.si/libs/lib-jitsi-meet.min.js'
    },
    options: {
      type: Object,
      default: () => ({
        disableSimulcast: true,
      }),
    },
    connectionOptions: {
      type: Object,
      default: () => ({})
    },
    conferenceName: {
      type: String,
      default: 'conference-1'
    },
    conferenceOptions: {
      type: Object,
      default: () => ({}),
    }
  },
  data () {
    return {
      jitsiApi: null,
      jitsiConnection: null,
      jitsiRoom: null,
    }
  },
  mounted () {
    this.loadScript(this.libraryUrl, () => {
      if (!window.JitsiMeetJS) throw new Error('Jitsi Meet API not loaded');
      this.init();
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
    init () {
      this.jitsiApi = JitsiMeetJS.init(this.options);
      this.jitsiConnection = new this.jitsiApi.JitsiConnection(null, null, this.connectionOptions)
      
      this.jitsiConnection.addEventListener(this.jitsiApi.events.connection.CONNECTION_ESTABLISHED, this.onConnectionSuccess);
      this.jitsiConnection.addEventListener(this.jitsiApi.events.connection.CONNECTION_FAILED, this.onConnectionFailed);
      this.jitsiConnection.addEventListener(this.jitsiApi.events.connection.CONNECTION_DISCONNECTED, this.disconnect);

      this.jitsiConnection.connect();
    },
    // executeCommand (command, ...value) {
    //   this.jitsiApi.executeCommand(command, ...value);
    // },
    // addEventListener (event, fn) {
    //   this.jitsiApi.on(event, fn);
    // },
    // Internal Events
    onConnectionSuccess (event) {
      console.log('onConnectionSuccess', event);
      this.jitsiRoom = this.jitsiConnection.initJitsiConference(this.conferenceName, this.conferenceOptions);
      this.jitsiRoom.on(this.jitsiApi.events.conference.TRACK_ADDED, this.onRemoteTrack);
      this.jitsiRoom.on(this.jitsiApi.events.conference.CONFERENCE_JOINED, this.onConferenceJoined);

      // await JitsiMeetJS.createLocalTracks();

      this.jitsiRoom.join();
    },
    onConnectionFailed (event) {
      console.log('onConnectionFailed', event);
    },
    disconnect (event) {
      console.log('disconnect', event);
    },
    onRemoteTrack (event) {
      console.log('onRemoteTrack', event);
    },
    onConferenceJoined (event) {
      console.log('onConferenceJoined', event);
    },
  }
};
</script>
