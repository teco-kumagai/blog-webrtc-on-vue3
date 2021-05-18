<template>
  <div class="select-root">
    <div class="select-container">
      <label>音声入力デバイス：</label>
      <select id="audioinput" @change="onSelectedValueChange">
        <option
          v-for="item in audioInput"
          :key="item.value"
          :value="item.value"
        >
          {{ item.label }}
        </option>
      </select>
    </div>
    <div class="select-container">
      <label>音声出力デバイス：</label>
      <select id="audiooutput" @change="onAudioOutputChange">
        <option
          v-for="item in audioOutput"
          :key="item.value"
          :value="item.value"
        >
          {{ item.label }}
        </option>
      </select>
    </div>
    <div class="select-container">
      <label>映像入力デバイス：</label>
      <select id="videoinput" @change="onSelectedValueChange">
        <option
          v-for="item in videoInput"
          :key="item.value"
          :value="item.value"
        >
          {{ item.label }}
        </option>
      </select>
    </div>
  </div>

  <video
    v-if="captureStream"
    :srcObject.prop="captureStream"
    id="video"
    autoplay
    playsinline
    muted
  ></video>
</template>

<script lang="ts">
/* global MediaStreamConstraints MediaDeviceKind */
import { computed, defineComponent, onMounted, ref } from "vue";

export default defineComponent({
  name: "MediaDevice",

  setup() {
    // ドロップダウンリストの項目に使用するデータのインターフェース
    interface SelectOption {
      label: string;
      value: string;
    }

    // ドロップダウンリストの項目に使用するデータを生成する関数
    const generateOptionValue = (kind: MediaDeviceKind): SelectOption[] =>
      deviceList.value
        .filter((device) => device.kind === kind)
        .map((filterdDevice) => ({
          label: filterdDevice.label,
          value: filterdDevice.deviceId,
        }));

    // コンポーネント内で使用する変数の定義
    const captureStream = ref<MediaStream | undefined>();
    const deviceList = ref<MediaDeviceInfo[]>([]);
    const audioInput = computed(() => generateOptionValue("audioinput"));
    const audioOutput = computed(() => generateOptionValue("audiooutput"));
    const videoInput = computed(() => generateOptionValue("videoinput"));

    // 取得したデバイス情報を変数に保存する関数
    const gotDevices = (devices: MediaDeviceInfo[]) =>
      (deviceList.value = devices);

    // getUserMedia で MediaStream が取得できた際に実行する関数
    const gotStream = (stream: MediaStream) => (captureStream.value = stream);

    // Promise の処理で reject された際に実行する関数
    const handleError = (error: Error) => {
      console.error(
        "navigator.MediaDevices method failed. ",
        error.message,
        error.name
      );
    };

    // 利用可能な音声入力、音声出力、映像入力デバイスを取得する関数
    const getDevices = async () => {
      return navigator.mediaDevices
        .enumerateDevices()
        .then(gotDevices)
        .catch(handleError);
    };

    // 既に MediaStream が存在している場合、その中の MediaStreamTrack をストップする関数
    const stopTracks = () => {
      if (captureStream.value) {
        captureStream.value.getTracks().forEach((track) => track.stop());
      }
    };

    // ドロップダウンリスト で選択されている値を取得する関数
    const getSelectedValue = (kind: MediaDeviceKind) => {
      const element = document.getElementById(kind);
      if (element instanceof HTMLSelectElement) {
        return element.value;
      }
    };

    // 映像表示を開始する関数
    const start = async () => {
      stopTracks();
      const selectedAudioSource = getSelectedValue("audioinput");
      const selectedVideoSource = getSelectedValue("videoinput");
      // getUserMedia の際に指定する、使用するデバイスの制約事項の定義
      const constraints: MediaStreamConstraints = {
        audio: {
          deviceId: selectedAudioSource
            ? { exact: selectedAudioSource }
            : undefined,
        },
        video: {
          deviceId: selectedVideoSource
            ? { exact: selectedVideoSource }
            : undefined,
        },
      };
      // 映像を取得する Web API のメソッドを実行
      return navigator.mediaDevices
        .getUserMedia(constraints)
        .then(gotStream)
        .catch(handleError);
    };

    // 音声出力デバイスを変更する関数
    const attachSinkId = () => {
      const selectedAudioOutput = getSelectedValue("audiooutput");
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      const videoElement = document.querySelector<any | null>("video");
      if (
        selectedAudioOutput &&
        videoElement &&
        videoElement.setSinkId !== "undefined"
      ) {
        videoElement
          .setSinkId(selectedAudioOutput)
          .then(() =>
            console.log(
              "音声出力デバイスを切り替えました。\n",
              "audiooutput:",
              selectedAudioOutput
            )
          )
          .catch(handleError);
      } else {
        console.warn(
          "このブラウザは音声出力デバイスの切り替えに対応していません。"
        );
      }
    };

    // ドロップダウンリストの項目が変更された際のハンドラ
    const onSelectedValueChange = async () => {
      await start();
      await getDevices();
    };

    // 音声出力デバイスのドロップダウンリストが変更された際のハンドラ
    const onAudioOutputChange = () => attachSinkId();

    // このコンポーネントが読み込まれた際に実行する初期処理
    onMounted(async () => {
      await getDevices();
      await start();
    });

    return {
      captureStream,
      audioInput,
      audioOutput,
      videoInput,
      onSelectedValueChange,
      onAudioOutputChange,
    };
  },
});
</script>

<style scoped>
.select-root {
  display: flex;
  flex-direction: column;
}

.select-container {
  display: inline-block;
  margin: 8px;
}
</style>
