# 一个简单的 Vue 头像上传器

这个简单朴素的 Vue 组件用于，帮助我们选取、裁剪图片作为头像。

## Getting Started

首先下载此组件:

```shell
npm install vue-plain-avatar-uploader
```

而后，使用之:

```vue
<template>
  <avatar-uploader />
</template>
<script>
import AvatarUploader from 'vue-plain-avatar-uploader';

export default {
  name: 'App',
  components: {
    AvatarUploader
  }
};
</script>
```

更详细的例子，请见 [这个仓库](https://github.com/taoqingqiu/vue-plain-avatar-uploader)。

## 组件 API

### 属性

| Name            | Type      | Default   | Description                                                                        |
| --------------- | --------- | --------- | ---------------------------------------------------------------------------------- |
| height          | `number`  | 150       | 组件的高                                                                           |
| maskProps       | `object`  | undefined | 定制化遮罩层的种种属性，详见[这里](#maskProps)                                     |
| outputProps     | `object`  | undefined | 定制化输出图片的属性，详见[这里](#outputProps)                                     |
| backgroundColor | `string`  | '#f2f3f8' | 放置图片的容器元素的背景色                                                         |
| maxMultiple     | `number`  | 2         | 图像相对于放置其的容器的最大放大倍数（初始时会有个自适应的过程，因为有所谓的放大） |
| hideShapeGroup  | `boolean` | undefined | 有时我们并不想切换用于切换遮罩层预览区域的形状，这时我们可以隐藏之                 |
| img             | `string`  | undefined | 此属性可为组件在初始时，预设置一个目标图片。需保证该值可以被 `img` 元素接受        |

### 插槽

#### action.innerSelectButton

当没有选择图片时，预览区域按说会有一个用来打开文件管理器、选择图片的按钮。

此插槽只提供一个 `click` 方法, 一个在 [vuetify](https://vuetifyjs.com) 中的示例用法如下:

```vue
<template>
  <avatar-uploader>
    <template #[`action.innerSelectButton`]="{ on }">
      <v-icon v-on="on">mdi-upload</v-icon>
    </template>
  </avatar-uploader>
</template>
```

#### action.slider

众所周知，slider 用于调节选取的目标图片的尺寸。示例用法：

```vue
<template>
  <avatar-uploader>
    <template #[`action.slider`]="{ on, attrs }">
      <v-slider v-on="on" v-bind="attrs" step="0.01" hide-details />
    </template>
  </avatar-uploader>
</template>
```

#### action.shapeGroup

所谓的 `shapeGroup` 是指用于切换遮罩层预览区域形状的 input 组，更具体地，应该是一对 radio。

```vue
<template>
  <avatar-uploader>
    <template #[`action.shapeGroup`]="{ on, attrs, options }">
      <v-radio-group @change="on.input" v-bind="attrs" row dense hide-details class="mt-0">
        <v-radio :label="option.text" :value="option.value" v-for="(option, index) in options" :key="index"></v-radio>
      </v-radio-group>
    </template>
  </avatar-uploader>
</template>
```

<small>\* 从示例中可以看出，vuetify 的 radio-group 双向绑定所使用的事件被魔改成了 `change`，这或许是符合 html 约定的 </small>

#### action.selectButton

与 `action.selectInnerButton` 功能上类似, 但此插槽的目标位置位于图片调整区域右侧。
另外，此插槽提供了个 `text` 属性，用于表征目标图片是否已选择。

```vue
<template>
  <avatar-uploader>
    <template #[`action.selectButton`]="{ on, text }">
      <v-btn v-on="on" small>{{ text }}</v-btn>
    </template>
  </avatar-uploader>
</template>
```

<small>\* 目前 text 值被粗糙地设定为 '选择' 或者 '重选'</small>

#### action.confirmButton

当我们完成了调整，需要导出给父组件时，便用到了这个插槽。
这个插槽提供了一个 `click` 方法，用来触发上述的导出。更具体而言，该方法会先通过 canvas.toDataUrl 方法生成结果图片，而后 `emit` 该结果给父组件绑定的 `confirm` 方法。

```vue
<template>
  <avatar-uploader>
    <template #[`action.confirmButton`]="{ on, attrs }">
      <v-btn v-on="on" v-bind="attrs" small>确定</v-btn>
    </template>
  </avatar-uploader>
</template>
```

### 事件

| Name    | Description              |
| ------- | ------------------------ |
| confirm | 用于导出结果图片给父组件 |

### <span id="maskProps">MaskProps</span>

```js
{
    // 遮罩层的颜色
    // eg. '#00000025', 'rgba(255,255,255, 0.37)'
    shadowColor: string,
    // 遮罩层预览区域的 border-radius
    // 默认 '5px'
    // 只有在遮罩层预览区域形状为方型时生效
    squareBorderRadius: string,
}
```

### <span id="outputProps">OutputProps</span>

```js
{
    // 输出图片的尺寸
    // 默认是 100x100 即 [100, 100]
    outputSize: array<number>,
    // 输出图片的格式
    // 默认是 'png'
    // eg. 'png', 'jpeg'
    format: string,
    // 当选中的图片区域只占预览区域的一部分时，
    // 输出图片会出现背景，
    // 此属性用于定制该背景色
    // 默认是 'transparent' ('png' 格式时) 和 '#fff' (其他格式)
    // 此外，传入 'transparent' 也只会在 'png' 格式时生效
    backgroundColor: string,
}
```

## Vuetify 组件中通过插槽定制化示例

为了更好展现如何定制化，一例以蔽之：

```vue
<template>
  <avatar-uploader @confirm="confirmAvatar" hideShapeGroup :img="avatar">
    <template #[`action.innerSelectButton`]="{ on }">
      <v-icon v-on="on">mdi-upload</v-icon>
    </template>
    <template #[`action.slider`]="{ on, attrs }">
      <v-slider v-on="on" v-bind="attrs" step="0.01" hide-details />
    </template>
    <template #[`action.shapeGroup`]="{ on, attrs, options }">
      <v-radio-group @change="on.input" v-bind="attrs" row dense hide-details class="mt-0">
        <v-radio :label="option.text" :value="option.value" v-for="(option, index) in options" :key="index"></v-radio>
      </v-radio-group>
    </template>
    <template #[`action.selectButton`]="{ on, text }">
      <v-btn v-on="on" small>{{ text }}</v-btn>
    </template>
    <template #[`action.confirmButton`]="{ on, attrs }">
      <v-btn v-on="on" v-bind="attrs" small>确定</v-btn>
    </template>
  </avatar-uploader>
</template>
<script>
export default {
  components: {
    AvatarUploader: () => import('vue-plain-avatar-uploader')
  }
  data() {
    return {
      avatar: 'xxx.png'
    };
  },
};
</script>
```

其效果可见本 REPO README 的 GIF 示范。
