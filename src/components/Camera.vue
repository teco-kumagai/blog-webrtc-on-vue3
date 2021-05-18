<template>
  <div class="root">
    <video
      v-if="captureStream"
      :srcObject.prop="captureStream"
      id="video"
      autoplay
      playsinline
      muted
    ></video>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "Camera",

  setup() {
    const captureStream = ref<MediaStream | undefined>();

    /* global MediaStreamConstraints */
    // 使用するデバイスの制約を定めるオブジェクト
    const constraints: MediaStreamConstraints = {
      audio: true,
      video: true,
    };

    // getUserMedia で MediaStream が取得できた際に実行する関数
    const gotStream = (stream: MediaStream) => {
      captureStream.value = stream;
    };

    // Promise の処理で reject された際に実行する関数
    const handleError = (error: Error) => {
      console.error(error.name, error.message);
    };

    // MediaStream の取得処理を実行
    navigator.mediaDevices
      .getUserMedia(constraints)
      .then(gotStream)
      .catch(handleError);

    return { captureStream };
  },
});
</script>
