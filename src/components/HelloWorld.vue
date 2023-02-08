<template>
  <div>
    <h3>Chime</h3>
    <div class="row">
      <label>meetingResponse</label>
      <input :value="meetingResponseJSON" @input="pasteMeetingResponse" />
    </div>

    <div class="row">
      <label>attendeeResponse</label>
      <input :value="attendeeResponseJSON" @input="pasteAttendeeResponse" />
    </div>

    <div class="row">
      Error: {{ errorMsg }}
    </div>

    <div><b>meetingResponse from URL</b></div>
    <div class="row query-string">{{ meetingResponseFromURL }}</div>

    <div><b>attendeeResponse from URL</b></div>
    <div class="row query-string">
      {{ attendeeResponseFromURL }}
    </div>

    <button @click="handleButtonClicked" class="join-button">Join meeting</button>
    <audio v-show="chimeMeetingSession" controls id="chime-player"></audio>
  </div>
</template>

<script>
import {
  ConsoleLogger,
  DefaultDeviceController,
  DefaultMeetingSession,
  LogLevel,
  MeetingSessionConfiguration,
} from 'amazon-chime-sdk-js';

export default {
  name: 'HelloWorld',

  data() {
    return {
      chimeMeetingSession: null,
      meetingResponse: '',
      attendeeResponse: '',
      errorMsg: null,
    }
  },

  computed: {
    meetingResponseJSON() {
      return JSON.stringify(this.meetingResponse)
    },

    attendeeResponseJSON() {
      return JSON.stringify(this.attendeeResponse)
    },


    meetingResponseFromURL() {
      const searchParams = new URLSearchParams(location.search).get('meetingResponse')

      return searchParams
    },

    attendeeResponseFromURL() {
      const searchParams = new URLSearchParams(location.search).get('attendeeResponse')

      return searchParams
    }

  },



  created() {
    this.getSearchParams()
  },

  mounted() {


  },

  methods: {
    pasteMeetingResponse($event) {
      console.log($event.target.value)

      this.meetingResponse = JSON.parse($event.target.value)

    },

    pasteAttendeeResponse($event) {

      console.log($event.target.value)

      this.attendeeResponse = JSON.parse($event.target.value)
    },

    getSearchParams() {
      let loc = "http://localhost:8080/?"
      let atd = { "ExternalUserId": "5defc894-0efb-4012-8447-98489eeb8671", "AttendeeId": "9d00e5b5-f21f-b930-1af6-47577caa501e", "JoinToken": "OWQwMGU1YjUtZjIxZi1iOTMwLTFhZjYtNDc1NzdjYWE1MDFlOmFkNGY5YmFmLWMzZmUtNDUwZi05MGZlLTk0NDYxODljYzc5OA" }
      let mtd = { "MeetingId": "88958a4a-9de3-48c5-9933-6066b6520706", "ExternalMeetingId": "dolfan-development-02abfdc4-e0a8-4c6c-894d-cbed6199622c", "MediaPlacement": { "AudioHostUrl": "80be0e7dd72e9eff302ce0ab155163d7.k.m2.ue1.app.chime.aws:3478", "AudioFallbackUrl": "wss://haxrp.m2.ue1.app.chime.aws:443/calls/88958a4a-9de3-48c5-9933-6066b6520706", "ScreenDataUrl": "wss://bitpw.m2.ue1.app.chime.aws:443/v2/screen/88958a4a-9de3-48c5-9933-6066b6520706", "ScreenSharingUrl": "wss://bitpw.m2.ue1.app.chime.aws:443/v2/screen/88958a4a-9de3-48c5-9933-6066b6520706", "ScreenViewingUrl": "wss://bitpw.m2.ue1.app.chime.aws:443/ws/connect?passcode=null&viewer_uuid=null&X-BitHub-Call-Id=88958a4a-9de3-48c5-9933-6066b6520706", "SignalingUrl": "wss://signal.m2.ue1.app.chime.aws/control/88958a4a-9de3-48c5-9933-6066b6520706", "TurnControlUrl": "https://ccp.cp.ue1.app.chime.aws/v2/turn_sessions", "EventIngestionUrl": "https://data.svc.ue1.ingest.chime.aws/v1/client-events" }, "MediaRegion": "us-east-1" }

      loc + "meetingResponse=" + JSON.stringify(mtd) + "&" + "attendeeResponse=" + JSON.stringify(atd)


      try {
        this.meetingResponse = JSON.parse(window.atob(new URLSearchParams(location.search).get('meetingResponse')))
        this.attendeeResponse = JSON.parse(window.atob(new URLSearchParams(location.search).get('attendeeResponse')))
      } catch (error) {
        this.errorMsg = error
      }
    },

    async handleButtonClicked() {
      this.setupMeetingSession()

      await this.startingSession()
    },

    // 建立一個 chime 會議室
    setupMeetingSession() {
      const logger = new ConsoleLogger('MyLogger', LogLevel.INFO);
      const deviceController = new DefaultDeviceController(logger);

      try {
        // You need responses from server-side Chime API. See below for details.

        /* The response from the CreateMeeting API action */
        const meetingResponse = this.meetingResponse;

        /* The response from the CreateAttendee or BatchCreateAttendee API action */
        const attendeeResponse = this.attendeeResponse;

        const configuration = new MeetingSessionConfiguration(
          meetingResponse,
          attendeeResponse
        );

        this.chimeMeetingSession = new DefaultMeetingSession(
          configuration,
          logger,
          deviceController
        );
      } catch (error) {
        this.errorMsg = error
      }
    },

    // Starting a session 加入會議室 (接聽電話)
    async startingSession() {
      const audioElement = document.getElementById(
        'chime-player'
      );

      // 設定麥克風
      try {
        const audioInputs = await this.chimeMeetingSession.audioVideo.listAudioInputDevices();

        await this.chimeMeetingSession.audioVideo.startAudioInput(
          audioInputs[0].deviceId
        );


        this.chimeMeetingSession.audioVideo.bindAudioElement(audioElement);

        const observer = {
          audioVideoDidStart: () => {
            console.log('Started');
          },

          audioVideoDidStop: () => {
            console.log('Stoped');
          },
        };

        this.chimeMeetingSession.audioVideo.addObserver(observer);

        this.chimeMeetingSession.audioVideo.start();

      } catch (err) {
        this.errorMsg = err
      }


    },
  }
}
</script>


<style scoped>
.row {
  margin-bottom: 1em;
}

.row.query-string {
  overflow-wrap: break-word;
}

.join-button {
  display: block;
  margin: 1em auto;
  padding: 0.5em;
  font-size: 1em;
}
</style>
