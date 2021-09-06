<template>
  <div :style="{ height: actualHeight + 'px' }" class="container">
    <div
      class="range-adjuster"
      ref="adjuster"
      @mouseup="onMouseUp"
      :style="{
        'background-image': image ? `url(${image})` : 'unset',
        'background-size': `${bgImageSize[0]}px ${bgImageSize[1]}px`,
        'background-position': adjusterBgPosi,
        'background-color': adjusterBgColor
      }"
    >
      <div
        class="mask"
        v-if="maskDiameter"
        ref="mask"
        :style="{
          height: maskDiameter + 'px',
          width: maskDiameter + 'px',
          'border-radius':
            maskShape === 'circle' ? '50%' : maskRectBorderRadius,
          'box-shadow': maskBoxShadow
        }"
        @mousedown="onMouseDown"
      >
        <slot
          name="action.innerSelectButton"
          :on="{ click: toSelectImage }"
          v-if="!image"
        >
          <button @click="toSelectImage" v-if="!image">click to select</button>
        </slot>
      </div>
    </div>
    <div class="actions">
      <div class="action">
        <slot
          name="action.slider"
          :attrs="{
            min: minEnlargeTimes,
            max: maxEnlargeTimes,
            value: enlargeTimes,
            disabled: !image
          }"
          :on="{ input: (times) => (enlargeTimes = times) }"
        >
          <input
            type="range"
            style="margin: 0 12px"
            class="action action-size"
            :min="minEnlargeTimes"
            :max="maxEnlargeTimes"
            v-model="enlargeTimes"
            step="0.01"
            :disabled="!image"
          />
        </slot>
      </div>
      <div class="action" v-if="hideShapeGroup === undefined">
        <slot
          name="action.shapeGroup"
          :attrs="{
            value: maskShape,
            disabled: !image
          }"
          :options="[
            { value: 'circle', text: '圆形' },
            { value: 'square', text: '方形' }
          ]"
          :on="{ input: (shape) => (maskShape = shape) }"
        >
          <label class="action-shape">
            <input
              name="shape"
              :disabled="!image"
              type="radio"
              value="circle"
              checked
              v-model="maskShape"
              style="margin-right: 8px"
            />圆形
          </label>
          <label class="action-shape">
            <input
              name="shape"
              :disabled="!image"
              type="radio"
              value="square"
              v-model="maskShape"
              style="margin-right: 8px"
            />方形
          </label>
        </slot>
      </div>

      <div class="action">
        <slot
          name="action.selectButton"
          :text="selectorText"
          :on="{ click: toSelectImage }"
        >
          <button type="button" class="action-button" @click="toSelectImage">
            {{ selectorText }}
          </button>
        </slot>
        <slot
          name="action.confirmButton"
          :attrs="{ disabled: !image }"
          :on="{ click: confirm }"
        >
          <button
            type="button"
            class="action-button"
            @click="confirm"
            :disabled="!image"
          >
            确定
          </button>
        </slot>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: [
    "height",
    "maskProps",
    "outputProps",
    "backgroundColor",
    "maxMultiple",
    "hideShapeGroup",
    "img"
  ],
  data() {
    return {
      image: null,
      maskDiameter: 0,
      maskBoxShadowRadius: 0,
      maskTransparent: false,
      containerShape: [],
      bgImageSrcSize: [],
      bgImagePostion: [0, 0],
      bgImageSrcPostion: [0, 0],
      bgStartPos: [],
      enlargeTimes: 1,
      maskShape: "circle"
    };
  },
  computed: {
    actualHeight() {
      return this.height || 150;
    },
    selectorText() {
      return !this.image ? "选择" : "重选";
    },
    maskBoxShadowColor() {
      return (this.maskProps && this.maskProps.shadowColor) || "#00000025";
    },
    maskRectBorderRadius() {
      return (this.maskProps && this.maskProps.squareBorderRadius) || "5px";
    },
    maskBoxShadow() {
      return `${
        this.maskTransparent ? this.maskBoxShadowColor : "#fff"
      } 0 0 0 ${this.maskBoxShadowRadius}px`;
    },
    adjusterBgColor() {
      return this.backgroundColor || "#f2f3f8";
    },
    adjusterBgPosi() {
      return `${this.bgImagePostion[0]}px ${this.bgImagePostion[1]}px`;
    },
    maxEnlargeTimes() {
      return this.maxMultiple || 2;
    },
    minEnlargeTimes() {
      const xRatio = this.maskDiameter / this.bgImageSrcSize[0],
        yRatio = this.maskDiameter / this.bgImageSrcSize[1];
      return (xRatio > yRatio ? xRatio : yRatio) || 0.5;
    },
    bgImageSize() {
      return [
        this.bgImageSrcSize[0] * this.enlargeTimes,
        this.bgImageSrcSize[1] * this.enlargeTimes
      ];
    },
    distSize() {
      return (this.outputProps && this.outputProps.outputSize) || [100, 100];
    },
    distFormat() {
      return (this.outputProps && this.outputProps.format) || "png";
    },
    distBackgroundColor() {
      return (
        (this.outputProps && this.outputProps.backgroundColor) || "transparent"
      );
    }
  },
  mounted() {
    setTimeout(() => {
      // 宽高、img 初始化
      this.initialize();
      // 绑一个必要的 mouseUp 事件
      document.addEventListener("mouseup", this.onMouseUp);
    }, 200);
  },
  beforeDestroy() {
    document.removeEventListener("mouseup", this.onMouseUp);
  },
  methods: {
    onMouseDown(event) {
      if (this.image) {
        this.bgStartPos = [event.x, event.y];
        // mask 半透明
        this.maskTransparent = true;

        // move 事件
        this.$refs.adjuster.addEventListener("mousemove", this.onMouseMove);
      }
    },
    onMouseMove(event) {
      let deltaX = event.x - this.bgStartPos[0],
        deltaY = event.y - this.bgStartPos[1];

      // 更新 bg image 位置
      this.bgImagePostion = [
        this.bgImagePostion[0] + deltaX,
        this.bgImagePostion[1] + deltaY
      ];
      this.bgStartPos = [event.x, event.y];
    },
    onMouseUp() {
      if (this.image) {
        // reset mask
        this.maskTransparent = false;

        // remove move 事件
        this.$refs.adjuster.removeEventListener("mousemove", this.onMouseMove);
      }
    },
    toSelectImage() {
      const input = document.createElement("input");
      input.type = "file";
      input.accept = "image/*";
      input.click();
      input.onchange = (e) => {
        this.enlargeTimes = 1;

        const targetImage = e.target.files[0];
        targetImage && (this.image = URL.createObjectURL(targetImage));

        // adjust image to fit container
        const img = document.createElement("img");
        img.src = this.image;
        img.onload = this.imgOnload;
      };
    },
    imgOnload(e) {
      const { width, height } = e.target;
      const xRatio = this.containerShape[0] / width,
        yRatio = this.containerShape[1] / height;
      if (xRatio > yRatio) {
        this.bgImageSrcSize = [width * xRatio, height * xRatio];
        this.bgImagePostion = [
          0,
          (-height * xRatio + this.containerShape[1]) / 2
        ];
      } else {
        this.bgImageSrcSize = [width * yRatio, height * yRatio];
        this.bgImagePostion = [
          (-width * yRatio + this.containerShape[0]) / 2,
          0
        ];
      }
    },
    initialize() {
      const { offsetHeight, offsetWidth } = this.$refs.adjuster;

      // mask
      this.maskDiameter =
        offsetHeight < offsetWidth ? offsetHeight : offsetWidth;
      this.maskBoxShadowRadius =
        (Math.pow(offsetWidth ** 2 + offsetHeight ** 2, 0.5) -
          this.maskDiameter) /
        2;

      // adjuster
      this.containerShape = [offsetWidth, offsetHeight];

      // img
      if (this.img) {
        this.image = this.img;
        const img = document.createElement("img");
        img.src = this.img;
        img.onload = this.imgOnload;
      }
    },
    confirm() {
      const img = document.createElement("img");
      img.setAttribute("crossorigin", "anonymous");
      img.src = this.image;

      const vm = this;
      img.onload = function () {
        vm.cropImage(this);
      };
    },
    cropImage(img) {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      canvas.width = this.distSize[0];
      canvas.height = this.distSize[1];

      // background
      if (
        this.distBackgroundColor === "transparent" &&
        this.distFormat !== "png"
      ) {
        ctx.fillStyle = "#fff";
      } else {
        ctx.fillStyle = this.distBackgroundColor;
      }
      ctx.rect(0, 0, canvas.width, canvas.height);
      ctx.fill();

      // crop
      const { width } = img;
      const ratio = width / this.bgImageSize[0];
      ctx.drawImage(
        img,
        (-this.bgImagePostion[0] +
          (this.containerShape[0] - this.maskDiameter) / 2) *
          ratio,
        (-this.bgImagePostion[1] +
          (this.containerShape[1] - this.maskDiameter) / 2) *
          ratio,
        this.maskDiameter * ratio,
        this.maskDiameter * ratio,
        0,
        0,
        this.distSize[0],
        this.distSize[1]
      );
      this.$emit("confirm", canvas.toDataURL(`image/${this.distFormat}`));
    }
  }
};
</script>
<style scoped>
.container {
  display: flex;
  justify-content: space-between;
  background-color: white;
  padding: 12px;
}
.action {
  display: flex;
  width: 100%;
  justify-content: space-around;
  user-select: none;
  align-items: center;
}
.actions,
.range-adjuster {
  display: inline-flex;
  justify-content: center;
}
.actions {
  width: 37%;
  flex-wrap: wrap;
  align-content: space-around;
}
.action-button {
  font-size: 14px;
}
.action-shape {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
}
.range-adjuster {
  width: 60%;
  align-items: center;
  overflow: hidden;
  background-repeat: no-repeat;
  cursor: move;
}
.mask {
  transition: 0.37s ease-in;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
