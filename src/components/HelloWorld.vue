<script setup>
import { ref, onMounted, reactive } from 'vue'
import sha256 from 'crypto-js/sha256';
import md5 from 'crypto-js/md5';
// import hmacSHA512 from 'crypto-js/hmac-sha512';
import * as echarts from 'echarts';
defineProps({
  msg: String
})

const canvasRef = ref(null)
let canvasFinger = ref('')
let webglString = reactive({})
const navigatorObj = reactive(navigator)
// console.log('navigatorObj', navigatorObj)
const screenObj = reactive(screen)
// 获取时区信息
const timeObj = ref(new Date().getTimezoneOffset())
let audioFingerprint = ref('')
let audioFingerprintHash = ref('')

// 服务器协助返回信息
let extraJsonData = reactive({})
// 
let compositeString = ref('')
let compositeHash = ref('')


let showCanvasSample = ref(false)
function toggleCanvasSample() {
  showCanvasSample.value = !showCanvasSample.value
}


onMounted(() => {
  // DOM 元素将在初始渲染后分配给 ref
  console.log('canvaselement', canvasRef) // <div>This is a root element</div>
  const outScreenCanvas = document.createElement('canvas')
  const showCanvas = canvasRef.value
  const showCanvasCtx = showCanvas.getContext("2d")
  const canvas = outScreenCanvas
  const ctx = canvas.getContext("2d");

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
  canvasFinger.value = sha256(canvas.toDataURL())

  showCanvasCtx.drawImage(outScreenCanvas, 0, 0)
  // 获取webgl相关硬件信息
  const glCanvas = document.createElement('canvas')
  const gl = glCanvas.getContext("webgl2");
  const webglMsg = getHardwareInfo(gl)
  webglString.vendor = webglMsg.vendor
  webglString.renderer = webglMsg.renderer
  // console.log('webglString.value', webglString);
  
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

  test()
})
function test() {
  // const audioCtx = new AudioContext()
  // let oscillator= audioCtx.createOscillator()
  // let gainNode = audioCtx.createGain()
  // oscillator.connect(gainNode)
  // oscillator.connect(audioCtx.destination)
  // oscillator.type = 'sine'
  // oscillator.frequency.value = 196.00
  // gainNode.gain.setValueAtTime(0, audioCtx.currentTime)
  // gainNode.gain.linearRampToValueAtTime(1, audioCtx.currentTime + 0.01)
  // oscillator.start(audioCtx.currentTime)
  // gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1)
  // oscillator.stop(audioCtx.currentTime + 1)
  // console.log('oscillator', audioCtx.currentTime)
  var each = function(obj, iterator) {
    if (Array.prototype.forEach && obj.forEach === Array.prototype.forEach) {
        obj.forEach(iterator)
    } else if (obj.length === +obj.length) {
        for (var i = 0, l = obj.length; i < l; i++) {
            iterator(obj[i], i, obj)
        }
    } else {
        for (var key in obj) {
            if (obj.hasOwnProperty(key)) {
                iterator(obj[key], key, obj)
            }
        }
    }
  }

  var AudioContext = window.OfflineAudioContext || window.webkitOfflineAudioContext

  var context = new AudioContext(1, 44100, 44100)

  var oscillator = context.createOscillator()
  oscillator.type = 'triangle'
  oscillator.frequency.setValueAtTime(10000, context.currentTime)

  var compressor = context.createDynamicsCompressor()
  each([
    ['threshold', -50],
    ['knee', 40],
    ['ratio', 12],
    ['reduction', -20],
    ['attack', 0],
    ['release', 0.25]
  ], function (item) {
    if (compressor[item[0]] !== undefined && typeof compressor[item[0]].setValueAtTime === 'function') {
      compressor[item[0]].setValueAtTime(item[1], context.currentTime)
    }
  })

  oscillator.connect(compressor)
  compressor.connect(context.destination)
  oscillator.start(0)
  context.startRendering()

  var audioTimeoutId = setTimeout(function () {
    console.warn('Audio fingerprint timed out. Please report bug at https://github.com/Valve/fingerprintjs2 with your user agent: "' + navigator.userAgent + '".')
    context.oncomplete = function () { }
    context = null
    return done('audioTimeout')
  }, 100)

  context.oncomplete = (event) => {
    // var fingerprint
    console.log('fingerprintfingerprint', event.renderedBuffer.getChannelData(0))
    try {
      clearTimeout(audioTimeoutId)
      audioFingerprint.value = event.renderedBuffer.getChannelData(0)
        .slice(4500, 5000)
        .reduce(function (acc, val) { return acc + Math.abs(val) }, 0)
        .toString()
      oscillator.disconnect()
      compressor.disconnect()
      audioFingerprintHash.value = sha256(audioFingerprint.value)
      showChart(event.renderedBuffer.getChannelData(0).slice(4500, 5000))
    } catch (error) {
      console.log(error)
      return
    }
    console.log(audioFingerprint.value, 
      sha256(
        audioFingerprint.value
      )
      .toString()
    )
  }
}

function showChart(source) {
  const temparray = Array.from(source)
  const key = new Array(temparray.length).fill('')
  console.log('showChart', source, temparray, temparray.length)
  var myChart = echarts.init(document.getElementById('main'));
  // 绘制图表
  myChart.setOption({
    title: {
      text: 'audio 波形'
    },
    tooltip: {},
    dataZoom: [
      {
        type: 'inside',
        start: 0,
        end: 10
      },
      {
        start: 0,
        end: 10
      }
    ],
    xAxis: {
      data: key,
      type: 'category'
    },
    yAxis: {},
    series: [
      {
        data: temparray,
        type: 'line',
        // smooth: true
      }
    ]
  });
}
function getExtraJson() {
  $.getJSON('http://www.geoplugin.net/json.gp', function(data) {
    // let msgObj = JSON.stringify(data, null, 2)
    console.log('getJSON', data);
    // extraJsonData = reactive({
    //   ...data
    // })
    extraJsonData.geoplugin_city = data.geoplugin_city
    extraJsonData.geoplugin_timezone = data.geoplugin_timezone
    extraJsonData.geoplugin_request = data.geoplugin_request
  });
}
getExtraJson()
function getCompositeHash() {
  // 
  let tempCompositeObj = [
    webglString.vendor,
    webglString.renderer,
    navigatorObj.platform,
    navigatorObj.maxTouchPoints,
    navigatorObj.hardwareConcurrency,
    `${ screenObj.width }*${ screenObj.height }*${ screenObj.colorDepth }`,
    timeObj.value,
    canvasFinger.value,
    audioFingerprint.value
  ]
  let tempcompositeString = ``
  for(let i=0;i<tempCompositeObj.length;i++) {
    if(i === 0) {
      tempcompositeString = `${tempCompositeObj[i]}`
    } else {
      tempcompositeString += `_${tempCompositeObj[i]}`
    }
  }
  console.log('compositeString', tempcompositeString)
  compositeString.value = tempcompositeString
  compositeHash.value = sha256(compositeString.value)
}
</script>

<template>
  <!-- <h1>{{ msg }}</h1> -->
  <canvas id="canvas" ref="canvasRef"></canvas>
  <img v-show="showCanvasSample" alt="canvas sample" src="../assets/canvas-fingerprinting.gif" />
  <div id="main" class="audio_chart"></div>
  <table border class="finger_table">
    <thead>
      <th>name</th>
      <th>value</th>
    </thead>
    <tbody>
      <tr>
        <td>canvas finger<button @click="toggleCanvasSample">toggle sample</button></td>
        <td>{{ canvasFinger }}</td>
      </tr>
      <tr>
        <td>webgl vendor</td>
        <td>{{ webglString.vendor }}</td>
      </tr>
      <tr>
        <td>webgl renderer</td>
        <td>{{ webglString.renderer }}</td>
      </tr>
      <tr>
        <td>
          audio finger
          <!-- <button @click="test">get Audio fingerprint</button> -->
        </td>
        <td>{{ audioFingerprint }}({{ audioFingerprintHash }})</td>
      </tr>
      <tr>
        <td>浏览器http请求中的用户代理</td>
        <td>{{ navigatorObj.userAgent }}</td>
      </tr>
      <tr>
        <td>浏览器的语言</td>
        <td>{{ navigatorObj.language }}</td>
      </tr>
      <tr>
        <td>操作系统</td>
        <td>{{ navigatorObj.platform }}</td>
      </tr>
      <tr>
        <td>设备能够支持的最大同时触摸的点数</td>
        <td>{{ navigatorObj.maxTouchPoints }}</td>
      </tr>
      <tr>
        <td>可用的逻辑处理器核心数</td>
        <td>{{ navigatorObj.hardwareConcurrency }}</td>
      </tr>
      <!-- <tr>
        <td>地理位位置信息</td>
        <td>{{ position }}</td>
      </tr> -->
      <tr>
        <td>浏览器插件</td>
        <td>{{ Array.from(navigatorObj.plugins).map(item => item.name).join(',') }}</td>
      </tr>
      <tr>
        <td>设备屏幕的宽高与色彩信息</td>
        <td>{{ screenObj.width }} * {{ screenObj.height }} * {{ screenObj.colorDepth }}</td>
      </tr>
      <tr>
        <td>格林威治时间和本地时间之间的时差</td>
        <td>{{ timeObj }}</td>
      </tr>
      <tr>
        <td>时区所属</td>
        <td>{{ extraJsonData.geoplugin_timezone }}</td>
      </tr>
      <tr>
        <td>地区</td>
        <td>{{ extraJsonData.geoplugin_city }}</td>
      </tr>
      <tr>
        <td>IP</td>
        <td>{{ extraJsonData.geoplugin_request }}</td>
      </tr>
      <tr>
        <td>compositeString</td>
        <td>{{ compositeString }}</td>
      </tr>
      <tr>
        <td><button @click="getCompositeHash">compositeHash</button></td>
        <td>{{ compositeHash }}</td>
      </tr>
    </tbody>
  </table>
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
.finger_table td,th {
  padding: 10px;
}
tr td:nth-child(1),th:nth-child(1) {
  text-align: right;
}
tr td:nth-child(2),th:nth-child(2) {
  text-align: left;
}
.audio_chart {
  width: 80%;
  height: 500px;
}
</style>
