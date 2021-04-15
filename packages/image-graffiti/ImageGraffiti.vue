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
            @mousedown='onmousedown'
            v-show="originImg"
            :class="{
              'text-mode':mode === 'text',
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
            top:textBoxRect.top+"px",
            color:fontBaseConfig.fillStyle,
            fontSize:fontSize
          }'
        ></div>
        <ul class="handle-list" v-show="originImg">
            <li
              class="base-btn text-edit"
              @click="handleLine"
              title="画笔"
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
            <li
              class="base-btn text-edit"
              @click="handleEdit"
              title="文字"
              :class="{
                active:mode === 'text'
              }"
            >
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
                    <el-color-picker v-model="fontBaseConfig.fillStyle" size="mini" show-alpha></el-color-picker>
                 </li>
               </ul>
            </li>
             <li class="base-btn" title='箭头'
              @click="handleArrow"
              :class="{
                active:mode === 'arrow'
              }"
            >
                <i class="el-icon-top-right"></i>
            </li>
             <li class="base-btn" title='撤销' @click="backHistory">
                <i class="el-icon-back"></i>
            </li>
            <li class="base-btn export" title='保存' @click='onExport'>
                <i class="el-icon-download"></i>
            </li>
        </ul>
    </div>
</template>

<script>
const CONST = {
  edgeLen: 50,
  angle: 25
}
const beginPoint = {};
const stopPoint = {};
const polygonVertex = [];
export default {
  name: 'ImageGraffiti',
  data () {
    return {
      originImg: null,
      canvas: null,
      ctx: null,
      lineWidth: 1,
      strokeStyle: 'rgba(255, 69, 0, 0.68)',
      mode: '', // line text arrow
      textBoxRect: null,
      textValue: '',
      fontSize: '24px',
      fontList: [
        {
          label: '小',
          value: '16px'
        },
        {
          label: '中',
          value: '20px'
        }, {
          label: '大',
          value: '24px'
        }
      ],
      fontBaseConfig: {
        textBaseline: 'top',
        fillStyle: 'rgba(255, 69, 0, 0.68)'
      },
      angle: '',
      history: [], // 历史记录列表
      currentHisory: 0, // 当前画布属于哪个history快照
      isDrawing: false,
      drawnSnapshot: null
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
    getImageData () {
      return this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    },
    setImageData (drawnSnapshot) {
      this.ctx.putImageData(drawnSnapshot, 0, 0);
    },
    recordSnapshot () {
      this.drawnSnapshot = this.getImageData();
    },
    drawSnapshot () {
      if (!this.drawnSnapshot) {
        return false;
      }
      console.log('绘制快照');
      this.setImageData(this.drawnSnapshot)
      return true;
    },
    pushHistory () {
      this.history.push(this.getImageData());
      const len = this.history.length;
      this.currentHisory = len - 1;
    },
    backHistory () {
      if (this.currentHisory > 0) {
        this.currentHisory -= 1;
      }
      this.history.length = this.currentHisory + 1;
      this.setImageData(this.history[this.currentHisory])
    },
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
        this.pushHistory();
      }
      image.src = url;
    },
    lmousemove (e) {
      const disX = e.clientX - this.canvas.offsetLeft;
      const disY = e.clientY - this.canvas.offsetTop;
      // 移动时设置画线的结束位置。并且显示
      this.ctx.lineTo(disX, disY) // 鼠标点下去的位置
      this.ctx.stroke()
    },
    lmouseup () {
      this.pushHistory();
      document.removeEventListener('mousemove', this.lmousemove);
      document.removeEventListener('mouseup', this.lmouseup);
    },
    amousemove (e) {
      this.drawSnapshot();
      const disX = e.clientX - this.canvas.offsetLeft;
      const disY = e.clientY - this.canvas.offsetTop;
      stopPoint.x = disX;
      stopPoint.y = disY;
      // alert(stopPoint.x+"+"+stopPoint.y);
      this.arrowCoord(beginPoint, stopPoint);
      this.sideCoord();
      this.drawArrow();
    },
    amouseup (e) {
      this.pushHistory();
      document.removeEventListener('mousemove', this.amousemove);
      document.removeEventListener('mouseup', this.amouseup);
    },
    onmousedown (e) {
      // 计算鼠标在画布的距离
      const disX = e.clientX - this.canvas.offsetLeft;
      const disY = e.clientY - this.canvas.offsetTop;
      this.recordSnapshot();
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
        document.addEventListener('mousemove', this.lmousemove);
        document.addEventListener('mouseup', this.lmouseup);
      } else if (this.mode === 'arrow') {
        beginPoint.x = disX;
        beginPoint.y = disY;
        this.ctx.moveTo(disX, disY);
        document.addEventListener('mousemove', this.amousemove);
        document.addEventListener('mouseup', this.amouseup);
      }
    },
    textValueInput (e) {
      this.textValue = e.target.innerText;
    },
    drawText () {
      this.ctx.moveTo(this.textBoxRect.left, this.textBoxRect.top);
      this.ctx.font = this.fontSize + ' 微软雅黑';
      const fontConfig = Object.entries(this.fontBaseConfig);
      fontConfig.forEach(([key, value]) => {
        this.ctx[key] = value;
      })
      this.ctx.fillText(this.textValue, this.textBoxRect.left, this.textBoxRect.top);
      this.textBoxRect = null;
      this.mode = '';
      this.textValue = '';
      this.pushHistory();
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
        this.textBoxRect = null;
      }
    },
    handleArrow () {
      if (this.mode === 'arrow') {
        this.mode = '';
      } else {
        this.mode = 'arrow';
        this.textBoxRect = null;
      }
    },
    onExport () {
      const url = this.canvas.toDataURL();
      const a = document.createElement('a');
      a.href = url;
      a.download = '导出';
      a.click();
    },
    // 箭头
    dynArrowSize: function () {
      var x = stopPoint.x - beginPoint.x;
      var y = stopPoint.y - beginPoint.y;
      var length = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2));
      if (length < 250) {
        CONST.edgeLen = CONST.edgeLen / 2;
        CONST.angle = CONST.angle / 2;
      } else if (length < 500) {
        CONST.edgeLen = CONST.edgeLen * length / 500;
        CONST.angle = CONST.angle * length / 500;
      }
      // console.log(length);
    },

    // getRadian 返回以起点与X轴之间的夹角角度值
    getRadian: function (beginPoint, stopPoint) {
      this.angle = Math.atan2(stopPoint.y - beginPoint.y, stopPoint.x - beginPoint.x) / Math.PI * 180;
      console.log(this.angle);
      this.paraDef(50, 25);
      this.dynArrowSize();
    },

    /// 获得箭头底边两个点
    arrowCoord: function (beginPoint, stopPoint) {
      polygonVertex[0] = beginPoint.x;
      polygonVertex[1] = beginPoint.y;
      polygonVertex[6] = stopPoint.x;
      polygonVertex[7] = stopPoint.y;
      this.getRadian(beginPoint, stopPoint);
      polygonVertex[8] = stopPoint.x - CONST.edgeLen * Math.cos(Math.PI / 180 * (this.angle + CONST.angle));
      polygonVertex[9] = stopPoint.y - CONST.edgeLen * Math.sin(Math.PI / 180 * (this.angle + CONST.angle));
      polygonVertex[4] = stopPoint.x - CONST.edgeLen * Math.cos(Math.PI / 180 * (this.angle - CONST.angle));
      polygonVertex[5] = stopPoint.y - CONST.edgeLen * Math.sin(Math.PI / 180 * (this.angle - CONST.angle));
    },

    // 获取另两个底边侧面点
    sideCoord: function () {
      var midpoint = {};
      // midpoint.x = polygonVertex[6] - (CONST.edgeLen * Math.cos(this.angle * Math.PI / 180));
      // midpoint.y = polygonVertex[7] - (CONST.edgeLen * Math.sin(this.angle * Math.PI / 180));
      midpoint.x = (polygonVertex[4] + polygonVertex[8]) / 2;
      midpoint.y = (polygonVertex[5] + polygonVertex[9]) / 2;
      polygonVertex[2] = (polygonVertex[4] + midpoint.x) / 2;
      polygonVertex[3] = (polygonVertex[5] + midpoint.y) / 2;
      polygonVertex[10] = (polygonVertex[8] + midpoint.x) / 2;
      polygonVertex[11] = (polygonVertex[9] + midpoint.y) / 2;
    },

    // 画箭头
    drawArrow: function () {
      const ctx = this.ctx;
      ctx.fillStyle = 'red';
      ctx.beginPath();
      ctx.moveTo(polygonVertex[0], polygonVertex[1]);
      ctx.lineTo(polygonVertex[2], polygonVertex[3]);
      ctx.lineTo(polygonVertex[4], polygonVertex[5]);
      ctx.lineTo(polygonVertex[6], polygonVertex[7]);
      ctx.lineTo(polygonVertex[8], polygonVertex[9]);
      ctx.lineTo(polygonVertex[10], polygonVertex[11]);
      // ctx.lineTo(polygonVertex[0], polygonVertex[1]);
      ctx.closePath();
      ctx.fill();
    },
    paraDef (edgeLen, angle) {
      CONST.edgeLen = edgeLen;
      CONST.angle = angle;
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
