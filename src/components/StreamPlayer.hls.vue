<template>
  <div>
    <div class="player-container">
      <div v-if="!buffered" class="player-loading-indicator">
        <div class="inline-block text-center">
          <i class="fas fa-spinner fa-spin"></i>
          <div>{{ playback ? 'loading' : 'requesting'}} media</div>
        </div>
      </div>
      <div v-if="playback" class="player-controls-wrapper">
        <div class="player-controls">
          <button class="player-control"
                  @click="toggleVolume">
            <i class="fa" 
               :class="{ 'fa-volume-down': controls.volume, 'fa-volume-off': !controls.volume }"></i>
          </button>
          <button class="player-control"
                  @click="toggleFullscreen">
            <i class="fa" 
               :class="{ 'fa-compress': controls.fullscreen, 'fa-expand': !controls.fullscreen }"></i>
          </button>
        </div>
      </div>
      <video id="player"/>
    </div>
  </div>
</template>

<script>
import StreamService from "../services/StreamService";
import FlussonicMsePlayer from "@flussonic/flussonic-mse-player";

const playerContainerId = "#preview-player";

let thumbsTempContainer;
export default {
  name: "StreamPlayer",
  props: ["stream"],
  mounted() {
    this.startPlayback();
  },
  destroyed() {
    this.scopeAlive = false;
    this.videoPlayer.stop();
  },
  data() {
    return {
      playback: false,
      buffered: true,
      controls: {
        volume: true,
        fullscreen: false
      },
      scopeAlive: true,
      flashDisabled: false,
      videoDOM: null,
      videoPlayer: null
    };
  },
  methods: {
    startPlayback() {
      const element = document.getElementById("player");
      const mbrControls = document.querySelector(".mbr-controls");
      this.videoDOM = element;

      let mediaProgressCount = 0;
      const playerConfig = {
        bufferMode: "sequence",
        onProgress: currentTime => {
          // if (++mediaProgressCount > 9) this.buffered = true
        },
        onMediaInfo: info => {
          this.playback = true;
        },
        onPlaybackError: () => {
          console.log('media error')
        },
        onError: e => {
          setTimeout(() => {
            if (window.player.stopPromise) {
              window.player.stopPromise.then(() => {
                window.player.setBufferMode("segments");
                window.player.play();
              });
            }
          }, 0);
        }
      };

      let playerInstance = new FlussonicMsePlayer(
        element,
        this.getStreamURL(),
        playerConfig
      );

      setTimeout(() => {
        this.buffered = false;
        playerInstance.play();
      }, 1000);

      window.player = this.videoPlayer = playerInstance;

      // DOM events
      element.onplaying = () => {
        this.buffered = true;
        // this.playback = true;
      };
    },
    getStreamURL() {
      const { region, key } = this.stream;
      let subdomain = region.identifierHaxr;
      let hostname = `${subdomain}.origincdn.com`;
      const url = `wss://${hostname}/${key}/mse_ld?tracks=v1a1`;
      // const url = `ws://${region.hostname}/${key}/mse_ld?tracks=v1a1`
      return url;
    },
    toggleVolume() {
      const oldVolume = this.videoDOM.volume;
      this.videoDOM.volume = oldVolume == 1 ? 0 : 1;

      this.controls.volume = this.videoDOM.volume === 1;
    },
    toggleFullscreen() {
      const elem = this.videoDOM;
      if (elem.requestFullscreen) {
        elem.requestFullscreen();
      } else if (elem.mozRequestFullScreen) {
        elem.mozRequestFullScreen();
      } else if (elem.webkitRequestFullscreen) {
        elem.webkitRequestFullscreen();
      }
    }
  }
};
</script>

<style>
.player-controls-wrapper {
  display: flex;
  align-items: flex-end;
  height: inherit;
  width: 100%;
  position: absolute;
  z-index: 1;
}
.player-controls {
  height: 40px;
  width: 100%;
  background-color: rgba(1, 3, 41, 0.55);
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  padding: 3px;
  transition: all 0.2s ease-in-out;
}
.player-controls-wrapper:hover .player-controls {
  background-color: rgba(1, 3, 41, 0.95);
}
.player-control {
  border: none;
  color: #ffffff;
  background-color: transparent;
  padding: 3px 14px;
  border-radius: 6px;
  outline: none !important;
  font-size: 18px;
  cursor: pointer;
}
.player-control:hover {
  background-color: #000;
}

#preview-player {
  margin: 10px 0;
  height: 100%;
}

#preview-player [data-player] {
  max-width: 100%;
}

#preview-player [data-no-op-msg] {
  font-size: 1.15em;
  padding: 0 10%;
  text-align: center;
}

#preview-player .player-poster {
  /* background-size: auto 100%; */
  background-size: 105% auto;
  border-radius: 4px;
}
#preview-player .container[data-container] {
  max-width: 100%;
}

.player-container {
  border: 1px solid black;
  height: inherit;
  position: relative;
}
#player {
  position: relative;
  width: 100%;
  max-width: 350px;
  max-height: 220px;
}
.mbr-controls {
  display: none;
}

div.player-error-screen__content[data-error-screen] {
  margin-top: 0;
  font-size: 12px;
}
div.player-error-screen__title[data-error-screen] {
  font-weight: 400;
  line-height: auto;
}
div.player-error-screen__message[data-error-screen] {
  text-transform: lowercase;
}
div.player-error-screen__code[data-error-screen] {
  display: none;
}
.player-loading-indicator {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: inherit;
  font-size: 11.5px;
}
.player-loading-indicator .fa-spin {
  font-size: 16px;
  margin-bottom: 6px;
}
</style>
