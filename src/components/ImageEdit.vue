<template>
    <div
    class="image-edit-wrap"
    :style="{
      width:width+'px',
      height:height+'px'
    }">
        <canvas
            ref='can'
            :width='width'
            :height="height"
            @mousedown='cmousedown'
            :class="{
              'text-mode':mode === 'text'
            }"
        ></canvas>
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
        <ul class="handle-list">
           <li class="base-btn" @click="handLineWidth(1)">
              <i class="el-icon-circle-plus-outline"></i>
            </li>
            <li class="base-btn" @click="handLineWidth(0)">
              <i class="el-icon-remove-outline"></i>
            </li>
            <li class="base-btn">
                <el-color-picker v-model="strokeStyle" show-alpha></el-color-picker>
            </li>
            <li class="base-btn" @click="handleEdit" :class="{
              active:mode === 'text'
            }">
               <i class="el-icon-edit-outline"></i>
               <ul class="text-style-handle-list">
                 <li class="base-text-style">
                 </li>
               </ul>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
  data () {
    return {
      canvas: null,
      ctx: null,
      lineWidth: 1,
      strokeStyle: 'rgba(255, 69, 0, 0.68)',
      mode: '',
      textBoxRect: null,
      textValue: ''
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
    this.initState();
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
    initState () {
      this.canvas = this.$refs.can;
      this.ctx = this.canvas.getContext('2d');
      const image = new Image();
      image.onload = () => {
        this.ctx.drawImage(image, 0, 0, this.width, this.height)
      }
      image.src = require('./timg.png');
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
        return
      };
      // 每次必须重新开始，让它们变成多个。
      this.ctx.beginPath()
      // 设置画线的宽，与颜色
      this.ctx.lineWidth = this.lineWidth;
      this.ctx.strokeStyle = this.strokeStyle;
      // 设置画的起始点
      this.ctx.moveTo(disX, disY)
      document.addEventListener('mousemove', this.cmousemove);
      document.addEventListener('mouseup', this.cmouseup);
    },
    textValueInput (e) {
      this.textValue = e.target.innerText;
    },
    drawText () {
      this.ctx.moveTo(this.textBoxRect.left, this.textBoxRect.top);
      this.ctx.fillStyle = 'purple';
      this.ctx.font = '20px "微软雅黑"';
      this.ctx.textBaseline = 'top';
      this.ctx.textAlign = 'left';
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
        }
    }
</style>
