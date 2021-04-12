<template>
    <div
      class="image-edit-wrap"
      :style="{
        width:width+'px',
        height:height+'px'
      }"
    >
        <canvas
            ref='can'
            :width='width'
            :height="height"
            @mousedown='cmousedown'
            v-show="originImg"
            :class="{
              'text-mode':mode === 'text'
            }"
        ></canvas>
        <div class="add-img" v-if="!originImg">
            <i class="el-icon-circle-plus-outline"></i>
            <input
              type="file"
              title="选择本地图片"
              @change="onChange"
              accept="image/*"
             >
        </div>
        <div
          class="text-box"
          v-if="textBoxRect"
          v-focus
          contenteditable
          @input="textValueInput"
          :style='{
            left:textBoxRect.left+"px",
            top:textBoxRect.top+"px"
          }'
        ></div>
        <ul class="handle-list" v-show="originImg">
            <li
              class="base-btn text-edit"
              @click="handleLine"
              :class="{
                active:mode === 'line'
              }"
            >
                <i class="el-icon-edit"></i>
                  <ul class="text-style-handle-list" v-if="mode ==='line'" @click.stop>
                    <li  @click="handLineWidth(1)">
                      <i class="el-icon-circle-plus-outline"></i>
                    </li>
                    <li  @click="handLineWidth(0)">
                      <i class="el-icon-remove-outline"></i>
                    </li>
                    <li>
                        <el-color-picker v-model="strokeStyle" show-alpha size="mini"></el-color-picker>
                    </li>
                </ul>
            </li>
            <li class="base-btn text-edit" @click="handleEdit" :class="{
              active:mode === 'text'
            }">
            T
               <ul class="text-style-handle-list" v-if="mode ==='text'" @click.stop>
                 <li class="select">
                     <el-select v-model="fontSize" placeholder="请选择" size="mini">
                        <el-option
                          v-for="item in fontList"
                          :key="item.value"
                          :label="item.label"
                          :value="item.value">
                        </el-option>
                     </el-select>
                 </li>
                 <li>
                    <el-color-picker v-model="fontColor" size="mini" show-alpha></el-color-picker>
                 </li>
               </ul>
            </li>
             <li class="base-btn" title='后退'>
                <i class="el-icon-back"></i>
            </li>
            <li class="base-btn export" title='导出' @click='onExport'>
                <i class="el-icon-download"></i>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
  data () {
    return {
      originImg: null,
      canvas: null,
      ctx: null,
      lineWidth: 1,
      strokeStyle: 'rgba(255, 69, 0, 0.68)',
      mode: '',
      textBoxRect: null,
      textValue: '',
      fontSize: 24,
      fontList: [
        {
          label: '小',
          value: 16
        },
        {
          label: '中',
          value: 20
        }, {
          label: '大',
          value: 24
        }
      ],
      fontColor: 'rgba(255, 69, 0, 0.68)',
      fontBaseConfig: {
        textBaseline: 'top',
        textAlign: 'left'
      }
    };
  },
  props: {
    width: {
      type: [Number, String],
      default: 500
    },
    height: {
      type: [Number, String],
      default: 500
    }
  },
  computed: {

  },
  created () {

  },
  mounted () {
    // this.initState();
  },
  directives: {
    focus: {
      inserted (el) {
        setTimeout(() => {
          el.focus();
        }, 100)
      },
      componentUpdated (el) {
        setTimeout(() => {
          el.focus();
        }, 100)
      }
    }
  },
  methods: {
    onChange (e) {
      const target = e.target;
      const file = target.files[0];
      target.value = '';
      const fileReader = new FileReader();
      fileReader.readAsDataURL(file);
      fileReader.onload = (evt) => {
        this.initState(evt.target.result)
        this.originImg = file;
      };
    },
    initState (url) {
      this.canvas = this.$refs.can;
      this.ctx = this.canvas.getContext('2d');
      const image = new Image();
      image.onload = () => {
        this.ctx.drawImage(image, 0, 0, this.width, this.height);
      }
      image.src = url;
    },
    cmousemove (e) {
      const disX = e.clientX - this.canvas.offsetLeft;
      const disY = e.clientY - this.canvas.offsetTop;
      // 移动时设置画线的结束位置。并且显示
      this.ctx.lineTo(disX, disY) // 鼠标点下去的位置
      this.ctx.stroke()
    },
    cmouseup () {
      document.removeEventListener('mousemove', this.cmousemove);
      document.removeEventListener('mouseup', this.cmouseup);
    },
    cmousedown (e) {
      // 计算鼠标在画布的距离
      const disX = e.clientX - this.canvas.offsetLeft;
      const disY = e.clientY - this.canvas.offsetTop;
      if (this.mode === 'text') {
        if (this.textValue) {
          this.drawText()
        } else {
          this.textBoxRect = {
            left: disX,
            top: disY
          }
        }
      } else if (this.mode === 'line') {
      // 每次必须重新开始，让它们变成多个。
        this.ctx.beginPath()
        // 设置画线的宽，与颜色
        this.ctx.lineWidth = this.lineWidth;
        this.ctx.strokeStyle = this.strokeStyle;
        // 设置画的起始点
        this.ctx.moveTo(disX, disY)
        document.addEventListener('mousemove', this.cmousemove);
        document.addEventListener('mouseup', this.cmouseup);
      }
    },
    textValueInput (e) {
      this.textValue = e.target.innerText;
    },
    drawText () {
      this.ctx.moveTo(this.textBoxRect.left, this.textBoxRect.top);
      this.ctx.fillStyle = this.fontColor;
      this.ctx.font = this.fontSize + 'px' + ' 微软雅黑';
      const fontConfig = Object.entries(this.fontBaseConfig);
      fontConfig.forEach(([key, value]) => {
        this.ctx[key] = value;
      })
      this.ctx.fillText(this.textValue, this.textBoxRect.left, this.textBoxRect.top);
      this.textBoxRect = null;
      this.mode = '';
      this.textValue = '';
    },
    handLineWidth (type) {
      if (type === 1) {
        this.lineWidth += 1;
      } else {
        if (this.lineWidth === 1) {
          return
        }
        this.lineWidth -= 1;
      }
    },
    handleEdit () {
      if (this.mode === 'text') {
        this.mode = '';
        this.textBoxRect = null;
      } else {
        this.mode = 'text';
      }
    },
    handleLine () {
      if (this.mode === 'line') {
        this.mode = '';
      } else {
        this.mode = 'line';
      }
    },
    onExport () {
      const url = this.canvas.toDataURL();
      const a = document.createElement('a');
      a.href = url;
      a.download = '导出';
      a.click();
    }
  },
  components: {

  }
};
</script>

<style lang="scss" scoped>
    .image-edit-wrap{
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        .add-img{
          width: 50px;
          height: 75px;
          border: 1px solid #ccc;
          border-radius: 5px;
          display: flex;
          justify-content: center;
          align-items: center;
          cursor: pointer;
          position: relative;
          input{
            font-size: 0;
            position: absolute;
            left: 0;
            top: 0;
            width: 50px;
            height: 75px;
            opacity: 0;
            cursor: pointer;
          }
        }
        .text-mode{
          cursor: text;
        }
        .text-box{
          min-width: 100px;
          min-height: 20px;
          border: 1px solid #333;
          position: absolute;
          outline: none;
          background-color: transparent;
        }
        .handle-list{
            position: absolute;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            justify-content: flex-start;
            align-content: center;
            list-style-type: none;
            height: 50px;
            background-color: rgba($color: #fff, $alpha: 0.5);
            border-radius: 10px 10px 0 0;
            padding: 0 20px;
            .base-btn{
              width: 50px;
              height: 100%;
              display: flex;
              justify-content: center;
              align-items: center;
              position: relative;
              cursor: pointer;
              &::after{
                position: absolute;
                content: '';
                right: 0;
                top: 50%;
                transform: translateY(-50%);
                height: 50%;
                width: 1px;
                background-color: #999;
              }
              &:last-child{
                 &::after{
                   display: none;
                 }
              }
              &:active{
                background-color: rgba($color: #333, $alpha: 0.1);
              }
              &.active{
                background-color: rgba($color: #333, $alpha: 0.1);
              }
            }
            .text-style-handle-list{
              position: absolute;
              bottom: 110%;
              left: 50%;
              transform: translateX(-50%);
              display: flex;
              justify-content: flex-start;
              align-items: center;
              background-color: rgba($color: #fff, $alpha: 0.5);
              border-radius: 10px;
              li{
                margin: 5px;
              }
              .select{
                width: 60px;
              }
            }
        }
    }
</style>
