<template>
  <div class="img-inputer" :class="[themeClass, sizeClass, nhe ? 'nhe': '', ]" ref="box">
    <i class="iconfont img-inputer__icon" v-html="iconUnicode"></i>
    <p class="img-inputer__placeholder">{{placeholder}}</p>

    <div class="img-inputer__preview-box" v-if="imgSelected">
      <img :src="dataUrl" class="img-inputer__preview-img">
    </div>
    <label :for="readonly ? '' : inputId" class="img-inputer__label"></label>

    <div class="img-inputer__mask" v-if="imgSelected && !noMask">
      <p class="img-inputer__file-name">{{fileName}}</p>
      <p class="img-inputer__change" v-if="readonly">{{readonlyTipText}}</p>
      <p class="img-inputer__change" v-else>{{bottomText}}</p>
    </div>

    <input
        ref="inputer"
        v-if="capture"
        type="file"
        :name="name"
        :id="inputId"
        :accept="accept"
        capture="video"
        class="img-inputer__inputer"
        @change="handleFileChange"
    />
    <input
        ref="inputer"
        v-else
        type="file"
        :name="name"
        :id="inputId"
        :accept="accept"
        class="img-inputer__inputer"
        @change="handleFileChange"
    />
    <transition name="vip-fade">
      <div class="img-inputer__err" v-if="errText.length">{{errText}}</div>
    </transition>
  </div>
</template>


<script type="text/ecmascript-6">
  export default {
    name: 'vue-img-inputer',
    props: {
      type: {
        default: 'img',
        type: String
      },

      accept: {
        default: 'image/*,video/*;',
        type: String
      },
      capture: {
        default: true,
        type: Boolean
      },
      id: {
        default: '',
        type: String
      },
      onChange: {
        default: null,
        type: Function
      },
      readonly: {
        type: Boolean,
        default: false
      },
      readonlyTipText: {
        default: 'Solo lectura',
        type: String
      },
      bottomText: {
        default: 'Haga clic o arrastre la imagen para editar',
        type: String
      },
      placeholder: {
        default: 'Haga clic o arrastre para seleccionar una imagen',
        type: String
      },
      value: {
        default: undefined
      },
      icon: {
        default: '',
        type: String
      },
      customerIcon: {
        default: '',
        type: String
      },
      maxSize: {
        default: 5120,
        type: Number
      },
      size: {
        default: '',
        type: String
      },
      imgSrc: {
        default: '',
        type: String
      },
      nhe: {
        type: Boolean,
        default: false
      },
      noMask: {
        type: Boolean,
        default: false
      },
      theme: {
        type: String,
        default: ''
      },
      name: {
        type: String,
        default: 'file'
      }
    },
    data () {
      return {
        inputId: '',
        file: [],
        dataUrl: '',
        fileName: '',
        errText: ''
      }
    },
    computed: {
      imgSelected () {
        return !!this.dataUrl || !!this.fileName;
      },
      sizeHumanRead () {
        let rst = 0;
        if (this.maxSize < 1024) {
          rst = this.maxSize + 'K';
        } else {
          rst = (this.maxSize / 1024).toFixed(this.maxSize % 1024 > 0 ? 2 : 0) + 'M';
        }
        return rst;
      },
      sizeClass () {
        if (this.size) {
          return `img-inputer--${this.size}`;
        }
      },
      themeClass () {
        return `img-inputer--${this.theme}`;
      },
      /**
       * @return {string}
       */
      ICON () {
        let rst = '';
        if (this.icon) {
          rst = this.icon;
        } else {
          rst = (this.theme === 'light' ? 'img' : 'clip');
        }
        return rst;
      },
      iconUnicode () {
        let iconMap = {
          img: '&#xe624;',
          clip: '&#xe62d;',
          img2: '&#xe62f;'
        };
        return this.customerIcon || iconMap[this.ICON]
      }
    },
    mounted () {
      this.inputId = this.id || this.gengerateID();
      if (this.imgSrc) {
        this.dataUrl = this.imgSrc;
      }
      ['dragleave', 'drop', 'dragenter', 'dragover'].forEach(e => {
        this.preventDefaultEvent(e);
      });
      this.addDropSupport();
    },
    methods: {
      preventDefaultEvent (eventName) {
        document.addEventListener(eventName, function (e) {
          e.preventDefault();
        }, false);
      },
      addDropSupport () {
        let BOX = this.$refs.box;
        BOX.addEventListener('drop', (e) => {
          e.preventDefault();
          if (this.readonly) return false;
          this.errText = '';
          let fileList = e.dataTransfer.files;
          if (fileList.length === 0) {
            return false;
          }
          if (fileList.length > 1) {
            this.errText = 'No es compatible con varios archivos';
            return false
          }
          this.handleFileChange(fileList);
        })
      },
      gengerateID () {
        let nonstr = Math.random().toString(36).substring(3, 8);
        if (!document.getElementById(nonstr)) {
          return nonstr;
        } else {
          return this.gengerateID();
        }
      },
      handleFileChange (e) {
        if (typeof e.target === 'undefined') this.file = e[0];
        else this.file = e.target.files[0];
        this.errText = '';
        let size = Math.floor(this.file.size / 1024);
        if (size > this.maxSize) {
          this.errText = `El tamaño del archivo no puede exceder ${this.sizeHumanRead}`;
          return false
        }
        this.$emit('input', this.file);
        this.onChange && this.onChange(this.file, this.file.name);
        this.$emit('onChange', this.file, this.file.name);
        this.imgPreview(this.file);
        this.fileName = this.file.name;
        this.resetInput();
      },
      imgPreview (file) {
        let self = this;
        if (!file || !window.FileReader) return;
        if (/^image/.test(file.type)) {
          let reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onloadend = function () {
            self.dataUrl = this.result;
          }
        }
      },
      resetInput () {
        let input = document.getElementById(this.inputId);
        let form = document.createElement('form');
        document.body.appendChild(form);
        let parentNode = input.parentNode;
        let isLastNode = parentNode.lastChild === input;
        let nextSibling;

        if (!isLastNode) {
          nextSibling = input.nextSibling;
        }
        form.appendChild(input);
        form.reset();
        isLastNode
          ? parentNode.appendChild(input)
          : parentNode.insertBefore(input, nextSibling);
        document.body.removeChild(form);
      }
    },
    watch: {
      imgSrc (newval) {
        this.dataUrl = newval;
        if (!newval) {
          this.file = [];
          this.errText = '';
          this.fileName = '';
        }
      },
      value (newval, oldval) {
        // Restablecer lógica
        if (oldval && !newval) {
          this.file = [];
          this.dataUrl = '';
          this.errText = '';
          this.fileName = '';
        }
      }
    }
  };
</script>

<style lang="scss">
  @import "../style/main.scss";
</style>
