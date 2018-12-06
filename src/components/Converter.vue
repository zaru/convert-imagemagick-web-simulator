<template>
  <div class="convert-simulator">
    <div class="params">
      <ul>
        <li>width <input type="number" v-model="options.width"></li>
        <li>height <input type="number" v-model="options.height"></li>
        <li>crop <input type="checkbox" v-model="options.crop"></li>
        <li>rotate <input type="number" v-model="options.rotate"></li>
        <li>flip <input type="checkbox" v-model="options.flip"></li>
        <li>flop <input type="checkbox" v-model="options.flop"></li>
        <li>unsharp <input type="checkbox" v-model="options.unsharp"></li>
        <li>file type <select v-model="options.ext">
          <option v-for="type in types" :value="type" :key="type">
            {{ type }}
          </option>
        </select></li>
        <li v-if="options.ext === 'jpg'">JPG quality <input type="number" v-model="options.quality"></li>
      </ul>
    </div>
    <textarea readonly @focus="select">$ {{command.join(' ')}}</textarea>
    <div class="example">
      <button @click="openDialog">upload</button>
      <input id="file" type="file" @change="loadImage" accept="image/*" hidden>
      <img id="outputImage">
      <img id="initializeImage" @load="setSize">
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import * as Magick from 'wasm-imagemagick'
import * as FileType from 'file-type'
export default {
  name: 'Converter',
  data () {
    return {
      options: {
        width: null,
        height: null,
        rotate: null,
        crop: null,
        flip: null,
        flop: null,
        unsharp: null,
        quality: null,
        ext: null
      },
      image: {
        ext: null,
        width: null,
        height: null,
        buffer: null
      },
      types: ['png', 'jpg', 'gif', 'webp']
    }
  },
  watch: {
    options: {
      handler () {
        this.im()
      },
      deep: true
    },
    'image.buffer' () {
      this.im()
    }
  },
  computed: {
    resize () {
      if (this.options.width || this.options.height) {
        if (this.options.crop) {
          return ['-resize', `${this.options.width||''}x${this.options.height||''}^`]
        }
        return ['-resize', `${this.options.width||''}x${this.options.height||''}`]
      }
    },
    crop () {
      if (this.options.crop) {
        return ['-gravity', 'center', '-crop', `${this.options.width}x${this.options.height}+0+0!`]
      }
    },
    quality () {
      if (this.options.ext === 'jpg' && this.options.quality) {
        return ['-quality', this.options.quality]
      }
    },
    rotate () {
      if (this.options.rotate) {
        return ['-rotate', this.options.rotate]
      }
    },
    flip () {
      if (this.options.flip) {
        return ['-flip']
      }
    },
    flop () {
      if (this.options.flop) {
        return ['-flop']
      }
    },
    unsharp () {
      if (this.options.unsharp) {
        return ['-unsharp', '12x6+0.5+0']
      }
    },
    command () {
      const options = []
      if (this.resize) {
        options.push(...this.resize)
      }
      if (this.crop) {
        options.push(...this.crop)
      }
      if (this.quality) {
        options.push(...this.quality)
      }
      if (this.rotate) {
        options.push(...this.rotate)
      }
      if (this.flip) {
        options.push(...this.flip)
      }
      if (this.flop) {
        options.push(...this.flop)
      }
      if (this.unsharp) {
        options.push(...this.unsharp)
      }
      return ['convert'].concat(options).concat([`src.${this.image.ext}`, `out.${this.options.ext}`])
    }
  },
  methods: {
    async im () {
      if (!this.image.ext) {
        return
      }
      const type = FileType(this.image.buffer)
      this.image.ext = type.ext
      const outputImage = document.getElementById('outputImage')
      const inputFiles = [{ name: `src.${type.ext}`, content: this.image.buffer }]
      const processedFiles = await Magick.Call(inputFiles, this.command)
      const firstOutputImage = processedFiles[0]
      outputImage.src = URL.createObjectURL(firstOutputImage["blob"])
    },
    setSize () {
      this.image.width = document.getElementById('initializeImage').width
      this.image.height = document.getElementById('initializeImage').height
    },
    loadImage (e) {
      const files = e.target.files || e.dataTransfer.files
      if (!files.length) {
        return
      }
      const reader = new FileReader()
      reader.addEventListener('load', () => {
        const buffer = new Uint8Array(reader.result)
        this.image.buffer = buffer
        this.setImage('initializeImage', buffer)
        this.setImage('outputImage', buffer)
      })
      reader.readAsArrayBuffer(files[0])
    },
    async loadExample () {
      const fetchedSourceImage = await fetch("/example.jpg");
      this.image.buffer = new Uint8Array(await fetchedSourceImage.arrayBuffer())
      this.setImage('initializeImage', this.image.buffer)
      this.setImage('outputImage', this.image.buffer)
    },
    setImage (id, buffer) {
      const initializeImage = document.getElementById(id)
      const blob = new Blob([buffer])
      initializeImage.src = URL.createObjectURL(blob)
      const type = FileType(buffer)
      this.image.ext = type.ext
      this.options.ext = type.ext
    },
    select (e) {
      e.target.select()
    },
    openDialog () {
      document.getElementById('file').click()
    }
  },
  async mounted () {
    await this.loadExample()
    this.im()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #initializeImage {
    display: none;
  }
  .convert-simulator {
    display: grid;
    height: 100vw;
    grid-template-rows: 1fr 60px 100vw;
    grid-template-columns: 1fr;
  }
  .example {
    text-align: center;
    background-color: #fff;
    background-image: linear-gradient(45deg, #c1c1c1 25%, transparent 25%, transparent 75%, #c1c1c1 75%, #c1c1c1), linear-gradient(45deg, #c1c1c1 25%, transparent 25%, transparent 75%, #c1c1c1 75%, #c1c1c1);
    background-position: 0 0, 5px 5px;
    background-size: 10px 10px;
  }
  .example input {
    margin: 0 0 10px 0;
  }
  .params {
    background-color: #3c3c3c;
    color: #d5d5d5;
    padding: 10px;
  }
  ul, li {
    margin: 0;
    padding: 0;
  }
  li {
    display: inline-block;
    padding: 0 10px;
    line-height: 2;
  }
  button {
    margin: 10px auto;
    padding: 5px 10px;
    border: none;
    border-radius: 5px;
    font-size: 14px;
    background-color: #3c3c3c;
    color: #d5d5d5;
    display: block;
  }
  input[type=file] {
    display: none;
  }
  input[type=number] {
    width: 50px;
    border: none;
    padding: 4px 5px;
    outline: none;
  }
  textarea {
    display: block;
    background: #333;
    color: #fafafa;
    width: 100%;
    box-sizing: border-box;
    padding: 10px;
    border: 0;
    font-size: 14px;
    outline: none;
  }
</style>
