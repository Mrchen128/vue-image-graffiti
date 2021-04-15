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
        <div
            class="add-img"
            v-if="!originImg"
            title="选择本地图片"
        >
            <svg t="1618495757116" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="10893" width="48" height="48"><path d="M778 183.8H247c-50.8 0-92 41.2-92 92v435c-1.2 4.8-1.2 9.4 0 14v23.6c0 50.8 41.2 92 92 92h531c50.8 0 92-41.2 92-92V275.8c0-50.8-41.2-92-92-92z m-531 56h531c19.8 0 36 16.2 36 36v349.4c-20.8-28.2-46-59-72.2-83.4-15.6-14.6-35-20.6-56.2-17.6-33.4 4.8-70 32.6-111.8 84.8-10 12.4-19 24.8-26.4 35.4-36.8-50.8-109.2-144.6-170.6-187.8-18.2-12.8-38.6-15.8-59.4-9-22.6 7.4-44.6 26.6-67 58.6-13 18.4-26.2 41.2-39.6 68V275.8c0.2-20 16.2-36 36.2-36z" p-id="10894" fill="#666666"></path><path d="M632 374.8a63 63 0 1 0 126 0 63 63 0 1 0-126 0z" p-id="10895" fill="#666666"></path></svg>
            <input
              type="file"
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
                <svg t="1618494265859" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4950" width="16" height="16"><path d="M173.013333 882.56l72.106667-203.733333L741.546667 132.266667 789.333333 129.92l76.373334 69.333333 2.346666 47.786667-496.64 546.773333z m109.44-181.333333l-35.84 101.76 99.2-44.586667 478.72-526.933333v-12.586667l-50.773333-45.44h-12.373333z" p-id="4951" fill="#666666"></path><path d="M678.229333 202.026667l126.336 114.752-28.693333 31.573333-126.336-114.730667z" p-id="4952" fill="#666666"></path></svg>
                  <ul class="text-style-handle-list" v-if="mode ==='line'" @click.stop>
                    <li  @click="handLineWidth(1)">
                      <svg t="1618494661105" class="icon" viewBox="0 0 1318 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="9501" width="16" height="16"><path d="M741.96894493 895.86428833l-89.12883988-217.48854506H317.69440381L232.08582869 895.86428833H164.38571167L446.0422962 200.63571166h74.87058778l288.82705529 695.22857666z m-402.82219204-267.39833244h292.3000683l-146.17366039-388.64663073zM737.68083754 513.40765239h321.91517757v42.12504211H737.68083754z" fill="#000" p-id="9502"></path><path d="M916.65910814 373.51849105v321.91517757h-42.1250421V373.51849105z" fill="#666666" p-id="9503"></path></svg>
                    </li>
                    <li  @click="handLineWidth(0)">
                      <svg t="1618494689476" class="icon" viewBox="0 0 1318 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="9839" width="16" height="16"><path d="M741.96894493 899.61428833l-89.12883989-217.48854506H317.69440381L232.08582869 899.61428833H164.38571167L446.0422962 204.38571166h74.87058778l288.8270553 695.22857666z m-402.82219204-267.39833244h292.3000683l-146.17366039-388.64663073zM737.68083754 550.0449541h321.91517757v42.12504212H737.68083754z" fill="#000" p-id="9840"></path></svg>
                    </li>
                    <li>
                        <!-- <el-color-picker v-model="strokeStyle" show-alpha size="mini"></el-color-picker> -->
                        <input type="color" v-model="strokeStyle">
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
              <svg t="1618494205109" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3335" width="16" height="16"><path d="M563.2 281.6V870.4a51.2 51.2 0 0 1-102.4 0V281.6H179.2a51.2 51.2 0 1 1 0-102.4h665.6a51.2 51.2 0 0 1 0 102.4H563.2z" fill="#666666" p-id="3336"></path></svg>
               <ul class="text-style-handle-list" v-if="mode ==='text'" @click.stop>
                 <li class="select">
                     <!-- <el-select v-model="fontSize" placeholder="请选择" size="mini">
                        <el-option
                          v-for="item in fontList"
                          :key="item.value"
                          :label="item.label"
                          :value="item.value"
                          >
                        </el-option>
                     </el-select> -->
                  <select v-model="fontSize">
                      <option
                          v-for="item in fontList"
                          :key="item.value"
                          :label="item.label"
                          :value="item.value"
                      >{{ item.value }}</option>
                  </select>
                 </li>
                 <li>
                    <!-- <el-color-picker v-model="fontBaseConfig.fillStyle" size="mini" show-alpha></el-color-picker> -->
                        <input
                          type="color"
                          v-model="fontBaseConfig.fillStyle"
                        >
                 </li>
               </ul>
            </li>
             <li class="base-btn" title='箭头'
              @click="handleArrow"
              :class="{
                active:mode === 'arrow'
              }"
            >
                <svg t="1618494136364" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="2136" width="16" height="16"><path d="M688.196267 356.272356 215.540622 175.069867c-18.386489-7.054222-36.443022 11.002311-29.3888 29.3888l181.202489 472.655644c5.893689 15.382756 25.691022 19.592533 37.341867 7.941689l62.828089-62.828089c8.886044-8.886044 23.290311-8.886044 32.176356 0l215.847822 215.847822c8.886044 8.886044 23.290311 8.886044 32.176356 0l102.172444-102.172444c8.886044-8.886044 8.886044-23.290311 0-32.176356L634.038044 487.901867c-8.886044-8.886044-8.886044-23.290311 0-32.176356l62.111289-62.111289C707.800178 381.963378 703.5904 362.177422 688.196267 356.272356z" p-id="2137" fill="#666666"></path></svg>
            </li>
             <li class="base-btn" title='撤销' @click="backHistory">
               <svg t="1618494309022" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="5850" width="16" height="16"><path d="M588.468659 257.265591H123.316451L371.227243 58.55359a31.947267 31.947267 0 1 0-39.614611-49.837737l-319.472671 255.578137v11.501016a30.669376 30.669376 0 0 0 0 4.472617v3.194727a30.669376 30.669376 0 0 0 0 4.472617v11.501016l319.472671 255.578137a31.947267 31.947267 0 1 0 40.253556-49.837737L123.316451 321.160125h465.152208C792.292223 321.160125 958.418011 464.283881 958.418011 640.632795s-166.125789 319.47267-369.949352 319.472671H95.841801a31.947267 31.947267 0 0 0 0 63.894534h492.626858C830.628943 1024 1022.312545 852.123703 1022.312545 640.632795s-191.683602-383.367205-433.843886-383.367204z" fill="#666666" p-id="5851"></path></svg>
            </li>
            <li class="base-btn export" title='保存' @click='onExport'>
              <svg t="1618494349867" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="6787" width="16" height="16"><path d="M916 293.6L730.4 108l-37.6-37.6c-4.1-4.1-9.7-6.4-15.6-6.4H166.5C109.9 64 64 109.9 64 166.4v691.1C64 914.1 109.9 960 166.4 960H857.5c56.6 0 102.4-45.9 102.4-102.4V346.7c0-5.8-2.3-11.4-6.4-15.6L916 293.6zM254.7 916V591.1c0-32.3 26.2-58.4 58.4-58.4h397.8c32.3 0 58.4 26.2 58.4 58.4V916H254.7zM916 857.6c0 32.3-26.2 58.4-58.4 58.4h-44.2V591.1c0-56.6-45.9-102.4-102.4-102.4H313.1c-56.6 0-102.4 45.9-102.4 102.4V916h-44.2c-32.3 0-58.4-26.2-58.4-58.4V166.4c0-32.3 26.2-58.4 58.4-58.4h44.2v150.6c0 56.6 45.9 102.4 102.4 102.4h324.6c12.2 0 22-9.8 22-22s-9.8-22-22-22H313.1c-32.3 0-58.4-26.2-58.4-58.4V108h404.4c5.8 0 11.4 2.3 15.6 6.4l234.9 234.9c4.1 4.1 6.4 9.7 6.4 15.6v492.7z" p-id="6788" fill="#666666"></path></svg>
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
      strokeStyle: '#ff0000',
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
        fillStyle: '#ff0000'
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
      const disX = e.pageX - this.canvas.offsetLeft;
      const disY = e.pageY - this.canvas.offsetTop;
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
      const disX = e.pageX - this.canvas.offsetLeft;
      const disY = e.pageY - this.canvas.offsetTop;
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
      const disX = e.pageX - this.canvas.offsetLeft;
      const disY = e.pageY - this.canvas.offsetTop;
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
          width: 80px;
          height: 100px;
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
            width: 80px;
            height: 100px;
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
                display: flex;
                select{
                  width: 100%;
                  outline: none;
                  height: 100%;
                  font-size: 16px;
                  cursor: pointer;
                }
              }
            }
        }
    }
</style>
