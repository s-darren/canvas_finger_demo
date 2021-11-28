<script setup>
import { ref, onMounted, reactive } from 'vue'
import sha256 from 'crypto-js/sha256';
// import hmacSHA512 from 'crypto-js/hmac-sha512';
defineProps({
  msg: String
})

const count = ref(0)
const canvasRef = ref(null)
let pngData = ref('')
let webglString = reactive({})
onMounted(() => {
  // DOM 元素将在初始渲染后分配给 ref
  console.log('canvaselement', canvasRef) // <div>This is a root element</div>
  const canvas = canvasRef.value
  const ctx = canvas.getContext("2d");;

  // // ctx.fillStyle = 'green';
  // // ctx.fillRect(10, 10, 150, 100);
  // // Text with lowercase/uppercase/punctuation symbols
  var txt = "BrowserLeaks,com <canvas> 1.0";
  ctx.textBaseline = "top";
  // The most common type
  ctx.font = "14px 'Arial'";
  ctx.textBaseline = "alphabetic";
  ctx.fillStyle = "#f60";
  ctx.fillRect(125,1,62,20);
  // Some tricks for color mixing to increase the difference in rendering
  ctx.fillStyle = "#069";
  ctx.fillText(txt, 2, 15);
  ctx.fillStyle = "rgba(102, 204, 0, 0.7)";
  ctx.fillText(txt, 4, 17);
  // console.log('canvas data', canvas.toDataURL())
  pngData.value = sha256(canvas.toDataURL())
  // pngData.value = hmacSHA512(canvas.toDataURL(), 'canvas finger')
  const glCanvas = document.createElement('canvas')
  const gl = glCanvas.getContext("webgl2");
  const webglMsg = getHardwareInfo(gl)
  webglString.vendor = webglMsg.vendor
  webglString.renderer = webglMsg.renderer
  console.log('webglString.value', webglString);
  function getHardwareInfo(gl) {
    var debugInfo = gl.getExtension("WEBGL_debug_renderer_info");
    if (!debugInfo) {
      return null;
    }
    var vendor = gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL);
    var renderer = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL);
    return {
      vendor: vendor,
      renderer: renderer,
    };
  }
})

</script>

<template>
  <h1>{{ msg }}</h1>
  <canvas id="canvas" ref="canvasRef"></canvas>
  <table border class="finger_table">
    <thead>
      <th>name</th>
      <th>value</th>
    </thead>
    <tbody>
      <tr>
        <td>canvas finger</td>
        <td>{{ pngData }}</td>
      </tr>
      <tr>
        <td>webgl vendor</td>
        <td>{{ webglString.vendor }}</td>
      </tr>
      <tr>
        <td>webgl renderer</td>
        <td>{{ webglString.renderer }}</td>
      </tr>
    </tbody>
  </table>
  <!-- <p class="png_data">HASH OF CANVAS FINGERPRINT: {{ pngData }}</p>
  <p class="png_data">vendor: {{ webglString.vendor }}</p>
  <p class="png_data">renderer: {{ webglString.renderer }}</p> -->
  <p>
    Recommended IDE setup:
    <a href="https://code.visualstudio.com/" target="_blank">VSCode</a>
    +
    <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
  </p>

  <p>
    <a href="https://vitejs.dev/guide/features.html" target="_blank">
      Vite Documentation
    </a>
    |
    <a href="https://v3.vuejs.org/" target="_blank">Vue 3 Documentation</a>
  </p>

  <button type="button" @click="count++">count is: {{ count }}</button>
  <p>
    Edit
    <code>components/HelloWorld.vue</code> to test hot module replacement.
  </p>
</template>

<style scoped>
a {
  color: #42b983;
}
/* .png_data {
  word-break: break-all;
} */
.finger_table {
  border: 2px solid #333333;
  border-collapse: collapse;
}
</style>
