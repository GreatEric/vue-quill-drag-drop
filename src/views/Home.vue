<template>
  <div>
    <div class="link-container">
      <router-link to="/QuillFormats">Quill Formats</router-link>
    </div>
    <div class="display-flex">
      <div class="div-size-left">
        <div
          v-for="(item, key) in optionList"
          :key="key"
          :id="'echarts' + key"
          class="echarts-div"
          draggable="true"
          @dragstart="dragstart($event, 'echarts' + key, 'canvas')"
          @dragend="dragend"
        ></div>
        <div class="div-drop-text" @drop="drop" @dragover.prevent>
          <p>{{this.dropData}}</p>
        </div>
        <div
          class="drag-div"
          draggable="true"
          @dragstart="dragstart($event, 'drag data', 'text')"
          @dragend="dragend"
        >
          <p>drag div</p>
        </div>
        <img
          id="img"
          src="../assets/images/1.jpg"
          alt="image"
          draggable="true"
          @dragstart="dragstart($event, 'image data', 'image')"
        >
        <canvas id="canvasHidden" v-show="false"></canvas>
        <div class="clear"></div>
        <br>
        <div id="table" draggable="true" @dragstart="dragstart($event, 'html canvas', 'html')">
          <md-table>
            <md-table-row>
              <md-table-head md-numeric>ID</md-table-head>
              <md-table-head>Name</md-table-head>
              <md-table-head>Email</md-table-head>
              <md-table-head>Gender</md-table-head>
              <md-table-head>Job Title</md-table-head>
            </md-table-row>

            <md-table-row>
              <md-table-cell md-numeric>1</md-table-cell>
              <md-table-cell>Shawna Dubbin</md-table-cell>
              <md-table-cell>sdubbin0@geocities.com</md-table-cell>
              <md-table-cell>Male</md-table-cell>
              <md-table-cell>Assistant Media Planner</md-table-cell>
            </md-table-row>

            <md-table-row>
              <md-table-cell md-numeric>2</md-table-cell>
              <md-table-cell>Odette Demageard</md-table-cell>
              <md-table-cell>odemageard1@spotify.com</md-table-cell>
              <md-table-cell>Female</md-table-cell>
              <md-table-cell>Account Coordinator</md-table-cell>
            </md-table-row>
          </md-table>
        </div>
      </div>
      <div class="div-size-right">
        <div class="quill-container">
          <quill-editor
            ref="quillEditor"
            v-model="content"
            :options="editorOption"
            id="editor"
          ></quill-editor>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import 'quill/dist/quill.snow.css'
import { quillEditor, Quill } from 'vue-quill-editor'
import { ImageDrop } from 'quill-image-drop-module'
import * as echarts from 'echarts'
import html2canvas from 'html2canvas'

export default {
  name: 'Home',
  data () {
    return {
      type: null,
      base64: null,
      content: '',
      editorOption: {},
      dropData: 'text drop',
      quill: null,
      optionList: [
        {
          xAxis: {
            data: ['ABC', 'DEF', 'GHI', 'JKL', 'MN', 'OPQ']
          },
          yAxis: {},
          series: [{
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
          }]
        },
        {
          xAxis: {
            data: ['God', 'ABC', 'DEF', 'HIJK', 'LM']
          },
          yAxis: {},
          series: [{
            type: 'line',
            data: [10, 20, 50, 80, 15]
          }]
        }
      ]
    }
  },
  components: {
    quillEditor
  },
  async mounted () {
    for (const key in this.optionList) {
      let e = echarts.init(document.getElementById('echarts' + key))
      e.setOption(this.optionList[key])
      e = null
    }

    Quill.register('modules/imageDrop', ImageDrop)
    this.quill = new Quill('#editor', { // eslint-disable-line
      theme: 'snow',
      modules: {
        imageDrop: true
      }
    })
    this.quill.root.addEventListener('drop', this.handleDrop, false)
    this.quill.root.addEventListener('dragover', this.handleDragover, false)

    // add change event
    function handler () {
      console.log('text-change')
    }
    this.quill.on('text-change', handler)

    // clear contents
    this.quill.setContents([])

    // insert text
    this.quill.insertText(0, 'Hello World', 'bold', true)
  },
  methods: {
    dragend (event) {
      event.dataTransfer.clearData()
    },
    handleDragover (evt) {
      evt.preventDefault()
      if (document.caretRangeFromPoint) {
        const selection = document.getSelection()
        const range = document.caretRangeFromPoint(evt.clientX, evt.clientY)
        if (selection && range) {
          selection.setBaseAndExtent(range.startContainer, range.startOffset, range.startContainer, range.startOffset)
        }
      }
    },
    async handleDrop (evt) {
      evt.preventDefault()
      if (evt.dataTransfer.files && evt.dataTransfer.files.length) {
        console.log('files is true')
      } else {
        if (this.type === 'canvas') {
          this.insert(this.base64)
        } else if (this.type === 'image') {
          let img = document.getElementById('img')
          let canvas = document.getElementById('canvasHidden')
          canvas.width = img.width
          canvas.height = img.height
          let ctx = canvas.getContext('2d')
          ctx.drawImage(img, 0, 0)
          let base64 = canvas.toDataURL()
          this.base64 = base64
          this.insert(this.base64)
        } else if (this.type === 'html') {
          let table = document.getElementById('table')
          const options = {
            type: 'dataURL'
          }
          await new Promise((resolve, reject) => {
            html2canvas(table, options).then((canvas) => {
              let base64 = canvas.toDataURL()
              this.base64 = base64
              resolve(true)
            })
          })
          this.insert(this.base64)
        } else {
          const index = (this.quill.getSelection() || {}).index || this.quill.getLength()
          this.quill.insertText(index, 'drag data text', {
            'color': '#a3a3a3',
            'italic': true
          })
        }
      }
    },
    insert (dataUrl) {
      const index = (this.quill.getSelection() || {}).index || this.quill.getLength()
      this.quill.insertEmbed(index, 'image', dataUrl, 'user')
    },
    readFiles (files, callback) {
      console.log('files: ', files)
    },
    drop (event) {
      let data = event.dataTransfer.getData('item')
      this.dropData = data
    },
    dragstart (event, data, type) {
      this.type = type
      this.base64 = null
      if (type === 'image') {
      } else if (type === 'canvas') {
        let canvas = document.getElementById(data).getElementsByTagName('canvas')[0]
        let base64 = canvas.toDataURL()
        this.base64 = base64
      } else {
        event.dataTransfer.setData('item', data)
      }
    }
  }
}
</script>
<style scoped="true">
.link-container {
  font-size: 1.6em;
  margin: 40px;
}
.display-flex {
  display: flex;
  height: 850px;
}
.div-size-left {
  width: 60%;
  margin: 10px;
}
.div-size-right {
  width: 40%;
  margin: 10px;
}
.quill-container {
  width: 100%;
  height: 100%;
}
.div-drop-text {
  border: 1px solid red;
  height: 200px;
  width: 150px;
  float: left;
}
.echarts-div {
  width: 300px !important;
  height: 260px;
  float: left;
}
.drag-div {
  border: 1px solid red;
  height: 200px;
  width: 150px;
  float: left;
  margin-left: 100px;
}
.clear {
  clear: both;
}
</style>
