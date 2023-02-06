<template>
  <div>
    <h3>Chime</h3>
    <div class="row">
      <label>meetingResponseFromQuery</label>
      <input v-mode="meetingResponseFromQuery"/>
    </div>

    <div class="row">
      <label>attendeeResponseFromQuery</label>
      <input v-model="attendeeResponseFromQuery"/>
    </div>

    <button @click="handleButtonClicked">Join meeting</button>
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
      meetingResponseFromQuery: '',
      attendeeResponseFromQuery: '',
    }
  },



  created() {
    this.meetingResponseFromQuery = new URLSearchParams(location.search).get('meetingResponse')
    this.attendeeResponseFromQuery = new URLSearchParams(location.search).get('attendeeResponse')
  },

  mounted() {


  },

  methods: {
    async handleButtonClicked() {
      this.setupMeetingSession()

      await this.startingSession()
    },

    // 建立一個 chime 會議室
    setupMeetingSession() {
      const logger = new ConsoleLogger('MyLogger', LogLevel.INFO);
      const deviceController = new DefaultDeviceController(logger);

      // You need responses from server-side Chime API. See below for details.
      const meetingResponse = this.meetingResponseFromQuery; /* The response from the CreateMeeting API action */
      const attendeeResponse = this.attendeeResponseFromQuery; /* The response from the CreateAttendee or BatchCreateAttendee API action */
      const configuration = new MeetingSessionConfiguration(
        meetingResponse,
        attendeeResponse
      );

      this.chineMeetingSession = new DefaultMeetingSession(
        configuration,
        logger,
        deviceController
      );
    },

    // Starting a session 加入會議室 (接聽電話)
    async startingSession() {
      const audioElement = document.getElementById(
        'chime-player'
      );

      // 設定麥克風
      try {
        const audioInputs = await this.chineMeetingSession.audioVideo.listAudioInputDevices();

        await this.chineMeetingSession.audioVideo.chooseAudioInputDevice(
          audioInputs[0].deviceId
        );
      } catch (err) {
        console.log("🚀 ~ file: HelloWorld.vue:92 ~ startingSession ~ err", err)
      }

      this.chineMeetingSession.audioVideo.bindAudioElement(audioElement);

      const observer = {
        audioVideoDidStart: () => {
          console.log('Started');
        },

        audioVideoDidStop: () => {
          console.log('Stoped');
        },
      };

      this.chineMeetingSession.audioVideo.addObserver(observer);

      this.chineMeetingSession.audioVideo.start();


    },
  }
}
</script>


<style scoped>
.row {
  margin-bottom: 1em;
}

button {
  padding: 0.5em;
  font-size: 1em;
}
</style>
