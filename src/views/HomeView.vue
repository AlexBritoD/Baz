<template>
  <div class="scanner-container">
    <div v-show="!isLoading" ref="wrapper">
      <video poster="data:image/gif,AAAA" ref="scanner"></video>
      <div class="overlay-element" id="content"></div>
      <p>{{ decodificado }}</p>
      <pre><code id="result"></code></pre>
      <div>
        <a class="button" id="startButton">Start</a>
        <a class="button" id="resetButton">Reset</a>
      </div>
      <div id="sourceSelectPanel" style="display: none">
        <label for="sourceSelect">Change video source:</label>
        <select id="sourceSelect" style="max-width: 400px"></select>
      </div>
    </div>
  </div>
</template>

<script>
import { BrowserQRCodeSvgWriter } from "@zxing/browser";
import { Exception, BrowserMultiFormatReader } from "@zxing/library";
export default {
  name: "stream-barcode-reader",
  data() {
    return {
      isLoading: true,
      codeReader: new BrowserMultiFormatReader(),
      codeWriter: new BrowserQRCodeSvgWriter(),
      isMediaStreamAPISupported:
        navigator &&
        navigator.mediaDevices &&
        "enumerateDevices" in navigator.mediaDevices,
      decodificado: "",
      selectedDeviceId: "",
      fullscreen: true,
      wheight: window.innerHeight,
      wwidth: window.innerWidth,
    };
  },
  mounted() {
    if (!this.isMediaStreamAPISupported) {
      throw new Exception("Media Stream API is not supported");
      return;
    }
    this.start();
    this.$refs.scanner.oncanplay = (event) => {
      this.isLoading = false;
      this.$emit("loaded");
    };
  },
  beforeUnmount() {
    this.codeReader.reset();
  },
  methods: {
    start() {
      this.codeReader.decodeFromConstraints(
        {
          audio: false, 
          video: { facingMode: "environment"  },
          deviceId: { exact: this.selectedDeviceId },
          width: { min: 640, ideal: 1920 },
          height: { min: 400, ideal: 1080 },
          aspectRatio: { ideal: 1.7777777778 },
        },
        this.$refs.scanner,
        (result, err) => {
          if (result) {
            this.$emit("decode", result.text);
            this.$emit("result", result);
            // this.decodificado = navigator.mediaDevices
            //   .enumerateDevices()
            //   .then(function (result) {
            //     return result[2];
            //   });
            this.decodificado = this.selectedDeviceId;
            this.codeWriter.writeToDom("#result", result.text, this.wwidth, this.wheight);

            this.codeReader
              .listVideoInputDevices()
              .then((videoInputDevices) => {
                const sourceSelect = document.getElementById("sourceSelect");
                this.selectedDeviceId = videoInputDevices[0].deviceId;
                if (videoInputDevices.length >= 1) {
                  videoInputDevices.forEach((element) => {
                    const sourceOption = document.createElement("option");
                    sourceOption.text =
                      element.label +
                      "----" +
                      element.kind +
                      "----" +
                      element.groupId;
                    sourceOption.value = element.deviceId;
                    sourceSelect.appendChild(sourceOption);
                  });

                  sourceSelect.onchange = () => {
                    this.selectedDeviceId = sourceSelect.value;
                  };

                  const sourceSelectPanel =
                    document.getElementById("sourceSelectPanel");
                  sourceSelectPanel.style.display = "block";
                }

                document
                  .getElementById("startButton")
                  .addEventListener("click", () => {
                    this.codeReader.decodeFromVideoDevice(
                      this.selectedDeviceId,
                      this.$refs.scanner,
                      (result, err) => {
                        if (result) {
                          // console.log(result);
                          document.getElementById("result").textContent =
                            result.text;
                        }
                      }
                    );
                    // console.log(
                    //   `Started continous decode from camera with id ${this.selectedDeviceId}`
                    // );
                  });

                document
                  .getElementById("resetButton")
                  .addEventListener("click", () => {
                    this.codeReader.reset();
                    document.getElementById("result").textContent = "";
                    // console.log("Reset.");
                  });
              })
              .catch((err) => {
                // console.error(err);
              });
          }
        }
      );
    },
  },
};
</script>

<style scoped>
video {
  max-width: 100%;
  max-height: 100%;
}
.scanner-container {
  position: relative;
}
.overlay-element {
  position: absolute;
  top: 0;
  width: 100%;
  height: 99%;
  background: rgba(30, 30, 30, 0.5);
  -webkit-clip-path: polygon(
    0% 0%,
    0% 100%,
    20% 100%,
    20% 20%,
    80% 20%,
    80% 80%,
    20% 80%,
    20% 100%,
    100% 100%,
    100% 0%
  );
  clip-path: polygon(
    0% 0%,
    0% 100%,
    20% 100%,
    20% 20%,
    80% 20%,
    80% 80%,
    20% 80%,
    20% 100%,
    100% 100%,
    100% 0%
  );
}

.overlay-element:before,
.overlay-element:after,
.overlay-element > :first-child:before,
.overlay-element > :first-child:after {
  position: absolute;
  width: 80px;
  height: 80px;
  border-color: red; /* or whatever colour */
  border-style: solid; /* or whatever style */
  content: " ";
}
.overlay-element:before {
  top: 0;
  left: 0;
  border-width: 10px 0 0 10px;
  border-top-left-radius: 10px;
}
.overlay-element:after {
  top: 0;
  right: 0;
  border-width: 10px 10px 0 0;
}
.overlay-element > :first-child:before {
  bottom: 0;
  right: 0;
  border-width: 0 10px 10px 0;
}
.overlay-element > :first-child:after {
  bottom: 0;
  left: 0;
  border-width: 0 0 10px 10px;
}

.laser {
  width: 60%;
  margin-left: 20%;
  background-color: tomato;
  height: 1px;
  position: absolute;
  top: 40%;
  z-index: 2;
  box-shadow: 0 0 4px red;
  -webkit-animation: scanning 2s infinite;
  animation: scanning 2s infinite;
}
@-webkit-keyframes scanning {
  50% {
    -webkit-transform: translateY(75px);
    transform: translateY(75px);
  }
}
@keyframes scanning {
  50% {
    -webkit-transform: translateY(75px);
    transform: translateY(75px);
  }
}
pre {
  background-color: white;
}
</style>
