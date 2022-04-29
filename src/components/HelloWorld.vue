<template>
  <div class="container flex column">
    <header>
      <input type="text" v-model="confName" :disabled="isConnected" />
      <button @click="onConnect" v-if="!isConnected">연결하기</button>
      <button @click="onDisConnectConference" v-if="isConnected">연결해제하기</button>
      <button @click="onRecordingStart" v-if="isConnected && !isRecording">녹화시작</button>
      <button @click="onRecordingStop" v-if="isConnected && isRecording">녹화중지</button>
      <button @click="onCreateLocalScreen" v-if="isConnected && !localScreen">화면공유적용</button>
      <button @click="onDestroyLocalScreen" v-if="isConnected && localScreen">화면공유해제</button>

      <div style="display:inline-block;" v-if="isConnected">
        <select id="camaraSource">
          <option v-for="(item, index) in cameraDevices" :key="index" :value="item.deviceId">{{item.label}}</option>
        </select>
        <button @click="onChangeCamera">카메라변경</button>
      </div>

      <div style="display:inline-block;" v-if="isConnected">
        <select id="micSource">
          <option v-for="(item, index) in micDevices" :key="index" :value="item.deviceId">{{item.label}}</option>
        </select>
        <button @click="onMicChange">마이크변경</button>
      </div>

      <div style="display:inline-block;" v-if="isConnected">
        <select id="speakerSource">
          <option v-for="(item, index) in speakerDevices" :key="index" :value="item.deviceId">{{item.label}}</option>
        </select>
        <button @click="onSpeakerChange">스피커변경</button>
      </div>

    </header>
    <div class="flex row">
      <div class="media-list">
        <div class="media-block" v-if="localMedia">
          <div class="participant-id">{{myId}}</div>
          <div class="local-container" @click="onClickSpotlight"></div>
        </div>

        <div
          class="media-block"
          v-for="(item, i) in remoteParticipants"
          :key="i"
          :id="item.participant.id"
          @click="onClickSpotlight"
        >
          <div class="participant-id">{{item.participant.id}}</div>
          <voice-meter :level="item.level" />
          <remote-video v-for="(video, j) in item.videos" :key="j" autoplay :remoteVideo="video" />
        </div>
      </div>

      <div class="spotlight">
        <video id="spotlight-video" autoplay playsInline></video>
      </div>
    </div>
  </div>
</template>

<script>
import voiceMeter from './voiceMeter.vue';
import remoteVideo from './remoteVideo.vue';
import ConnectLive from '@connectlive/connectlive-web-sdk';

export default {
  name: 'HelloWorld',
  components: {
    voiceMeter,
    remoteVideo
  },
  data() {
    return {
      confName: 'icl2-test-room',
      conf: null,
      localMedia: null,
      localScreen: null,
      remoteParticipants: [],
      isConnected: false,
      myId: '',
      cameraDevices: [],
      micDevices: [],
      speakerDevices: [],
      audioCheckInterval: -1,
      isRecording: false
    };
  },
  methods: {
    async onConnect() {
      await this.onProvisioning();
      await this.onCreateLocalMedia();
      await this.onCreateConference();
      await this.conf.connect(this.confName);

      this.myId = this.conf.localParticipant.id;
      this.isConnected = true;
      this.conf.publish([this.localMedia]);

      this.audioCheckInterval = window.setInterval(()=>{
        const audioLevels = this.conf.getRemoteAudioLevels();
        audioLevels.remoteParticipants.forEach((audioParticipant)=>{
          const participant = this.remoteParticipants.find(participant => participant.participant.id === audioParticipant.remoteParticipant.id);

          let level = parseFloat(audioParticipant.level.toFixed(3));
          if(level > 0.15) {
            level = 0.15;
          }

          participant.level = level / 0.15 * 100;
        });
      }, 250);
    },
    async onProvisioning() {
      await ConnectLive.signIn({
        serviceId: '202200000000',
        serviceKey: '2022000000000000',
        secret: '2022000000000000',
      });
    },
    async onCreateLocalMedia() {
      this.localMedia = await ConnectLive.createLocalMedia({
        audio: true,
        video: true
      });

      this.$nextTick(async ()=>{
        const video = this.localMedia.video.attach();
        video.className = 'local-video';
        video.style.width = '100%';
        document.querySelector('.local-container').appendChild(video);

        this.cameraDevices = await this.localMedia.getCameraDevices();

        this.micDevices = await this.localMedia.getMicDevices();

        this.speakerDevices = await this.localMedia.getSpeakerDevices();
      });
    },
    async onCreateLocalScreen() {
      this.localScreen = await ConnectLive.createLocalScreen({
        audio: true,
        video: true
      });
      this.localScreen.video.setExtraValue('screen');

      const video = this.localScreen.video.attach();
      video.className = 'local-video';
      video.style.width = '100%';
      document.querySelector('.local-container').appendChild(video);

      this.conf.publish([this.localScreen]);
    },
    async onCreateConference() {
      this.conf = ConnectLive.createConference();

      this.conf.on('connected', (evt)=>{
        evt.remoteParticipants.forEach(async participant => {
          let remoteVideos = [];
          const unsubscribedVideos = participant.getUnsubscribedVideos();
          if (unsubscribedVideos.length) {
            const videoIds = unsubscribedVideos.map(video => video.getVideoId());
            remoteVideos = await this.conf.subscribe(videoIds);
          }

          this.remoteParticipants.push({
            participant: participant,
            videos: remoteVideos,
            level: 0
          });
        });
      });

      this.conf.on('participantEntered', (evt)=>{
        this.remoteParticipants.push({
          participant: evt.remoteParticipant,
          videos: [],
          level: 0
        });
      });

      this.conf.on('participantLeft', evt => {
        const index = this.remoteParticipants.findIndex(item => item.participant.id === evt.remoteParticipantId);
        this.remoteParticipants.splice(index, 1);
      });

      this.conf.on('remoteVideoPublished', async evt => {
        const remoteVideos = await this.conf.subscribe([evt.remoteVideo.videoId]);
        if (remoteVideos.length) {
          const participant = this.remoteParticipants.find(item => item.participant.id === evt.remoteParticipant.id);
          if(participant) {
            participant.videos = participant.videos.concat(remoteVideos);
          }
        }
      });

      this.conf.on('remoteVideoUnpublished', evt => {
        const participant = this.remoteParticipants.find(item => item.participant.id === evt.remoteParticipant.id);
          if(participant) {
            participant.videos = participant.videos.filter((remoteVideo)=>{
              return remoteVideo.videoId !== evt.remoteVideo.videoId;
            });
          }
      });
    },
    onDisConnectConference() {
      this.conf.disconnect();

      this.localMedia.stop();
      this.localMedia.video.detach();

      this.localMedia = null;

      if(this.localScreen) {
        this.localScreen.stop();
        this.localScreen.video.detach();
        this.localScreen = null;
      }

      this.isConnected = false;
      this.myId = '';
      this.remoteParticipants = [];
      window.clearInterval(this.audioCheckInterval);
      this.audioCheckInterval = -1;

      ConnectLive.signOut();
    },
    async onDestroyLocalScreen() {
      this.localScreen.video.detach();
      this.localScreen.stop();

      this.conf.unpublish([this.localScreen]);

      this.localScreen = undefined;
    },
    async onChangeCamera() {
      this.localMedia.video.detach();

      const camaraSource = document.getElementById('camaraSource');
      await this.localMedia.switchCamera(camaraSource.value);

      const video = this.localMedia.video.attach();
      video.className = 'local-video';
      video.style.width = '100%';
      document.querySelector('.local-container').appendChild(video);
    },
    async onMicChange() {
      const micSource = document.getElementById('micSource');
      await this.localMedia.switchMic(micSource.value);
    },
    async onSpeakerChange() {
      const speakerSource = document.getElementById('speakerSource');
      await this.conf.switchSpeaker(speakerSource.value);
    },
    onClickSpotlight(event) {
      const target = event.target;
      const remoteParticipant = this.conf.remoteParticipants.find(participant => participant.id === target.dataset.participantid);
      if (remoteParticipant) {
        const t = remoteParticipant.getVideo(parseInt(target.dataset.videoid));
        t.setQuality('h');
      }

      document.getElementById('spotlight-video').srcObject = target.srcObject;
    },
    async onRecordingStart() {
      await this.conf.unpublish([this.localMedia]);

      await this.conf.publish([this.localMedia], true);

      this.isRecording = true;
    },
    async onRecordingStop() {
      await this.conf.unpublish([this.localMedia]);

      await this.conf.publish([this.localMedia]);

      this.isRecording = false;
    }
  }
}
</script>

<style scoped>
.container {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background-color: #101523;
  color: #fff;
}

input {
  line-height: 1.5;
  padding: 8px;
  border: 1px solid hsl(0, 0%, 10%);
  color: #fff;
  background: hsl(0, 0%, 14%);
  transition: background-color 0.3s cubic-bezier(0.57, 0.21, 0.69, 1.25),
    transform 0.3s cubic-bezier(0.57, 0.21, 0.69, 1.25);
}

button {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 0;
  cursor: pointer;
  margin: 10px;
  padding: 10px;
}

video {
  cursor: pointer;  
}

.flex {
  display: flex;
  flex: 1;
  height: 100%;
}

.flex.column {
  flex-direction: column;
}

.flex.row {
  flex-direction: row;
}

header {
  min-height: 55px;
  padding: 3px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  text-align: left;
}

.spotlight {
  width: 100%;
  padding: 10px;
}

.spotlight video {
  width: 100%;
  height: 100%;
}

.media-list {
  width: 300px;
  padding: 10px;
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  overflow: auto;
}

.media-list .media-block {
  position: relative;
  margin-top: 5px;
  padding: 10px;
  border: 1px solid #c9c9c9;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 5px;
  cursor: pointer;
}

.participant-id {
  position: absolute;
  bottom: 5px;
  left: 5px;
  color: #fff;
  font-size: 12px;
}
</style>