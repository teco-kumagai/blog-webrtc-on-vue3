<template>
  <div class="select-root">
    <div class="select-container">
      <label>音声入力デバイス：</label>
      <select id="audioinput">
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
      <select id="audiooutput">
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
      <select id="videoinput">
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
</template>

<script lang="ts">
/* global MediaDeviceKind */
import { computed, defineComponent, ref } from "vue";

export default defineComponent({
  name: "DeviceSelection",

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
    const deviceList = ref<MediaDeviceInfo[]>([]);
    const audioInput = computed(() => generateOptionValue("audioinput"));
    const audioOutput = computed(() => generateOptionValue("audiooutput"));
    const videoInput = computed(() => generateOptionValue("videoinput"));

    // 取得したデバイス情報を変数に保存する関数
    const gotDevices = (devices: MediaDeviceInfo[]) => {
      console.log("取得されたデバイス情報\n", devices);
      deviceList.value = devices;
    };

    // Promise の処理で reject された際に実行する関数
    const handleError = (error: Error) => {
      console.error(error.name, error.message);
    };

    // 利用可能な音声入力、音声出力、映像入力デバイスを取得
    navigator.mediaDevices
      .enumerateDevices()
      .then(gotDevices)
      .catch(handleError);

    return { audioInput, audioOutput, videoInput };
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
