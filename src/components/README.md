# Vue Plain Avatar Uploader

This is a plain vue component used to help cropping image when choosing one image as avatar.

## Getting Started

Firstly, install this component:

```shell
npm install vue-plain-avatar-uploader
```

Secondly, import and use this component:

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

More detailed example is [here](https://github.com/taoqingqiu/vue-plain-avatar-uploader#readme).

## Component API

### Props

| Name            | Type      | Default   | Description                                                                              |
| --------------- | --------- | --------- | ---------------------------------------------------------------------------------------- |
| height          | `number`  | 150       | height of this component                                                                 |
| maskProps       | `object`  | undefined | customize mask above image, details are [below](#maskProps)                              |
| outputProps     | `object`  | undefined | customize mask above image, details are [below](#outputProps)                            |
| backgroundColor | `string`  | '#f2f3f8' | background color of container containing image                                           |
| maxMultiple     | `number`  | 2         | maximum multiple of enlarging image, the reference is container                          |
| hideShapeGroup  | `boolean` | undefined | sometimes, we don't need to change shape of preview area                                 |
| img             | `string`  | undefined | url that can be accepted by attribute 'src' of element 'img', to set an image in advance |

### Slots

#### action.innerSelectButton

When no image is selected, there should be a button used to open file manager in preview area of mask layer.

This slot only provides a function `click`, usage example in framework `[vuetify](https://vuetifyjs.com)` is below:

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

So called slider is used to adjust image size, and also example is below:

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

Used to change shape of preview area at mask layer, example is below:

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

<small>\* Instead of function 'input', component radio-group of vuetify use function 'change' to implement bidirectional binding</small>

#### action.selectButton

Similar with `action.selectInnerButton` functionally, but the position of this slot is at right side of component.
Specially, this slot provides an attribute `text`, which indicates whether an image has been selected or not.

```vue
<template>
  <avatar-uploader>
    <template #[`action.selectButton`]="{ on, text }">
      <v-btn v-on="on" small>{{ text }}</v-btn>
    </template>
  </avatar-uploader>
</template>
```

<small>\* Recklessly, value of attribute 'text' is one of '选择' and '重选'</small>

#### action.confirmButton

Finally, we finish adjusting and cropping image, and click one confirming button.
This slot provides a function `confirm`, which we will emit function 'confirm' injected by parent component after triggering.

```vue
<template>
  <avatar-uploader>
    <template #[`action.confirmButton`]="{ on, attrs }">
      <v-btn v-on="on" v-bind="attrs" small>确定</v-btn>
    </template>
  </avatar-uploader>
</template>
```

### Functions

| Name    | Description                                                                          |
| ------- | ------------------------------------------------------------------------------------ |
| confirm | export output to parent component as an url (created by function `canvas.toDataUrl`) |

### <span id="maskProps">MaskProps</span>

```js
{
    // color of mask layer, the default is '#00000025'
    // eg. '#00000025', 'rgba(255,255,255, 0.37)'
    shadowColor: string,
    //border-radius of transparent area of mask
    // the default is '5px'
    // only works when shape of transparent area is set as 'square'
    squareBorderRadius: string,
}
```

### <span id="outputProps">OutputProps</span>

```js
{
    // size of output image
    // the default is [100, 100]
    outputSize: array<number>,
    // format of output image
    // the default is 'png'
    // eg. 'png', 'jpeg'
    format: string,
    // when selected part of image can not fill a square
    // we will see the background
    // the default is 'transparent' (when format is 'png') and '#fff' (other format)
    // nb. 'transparent' only works when 'png' format
    backgroundColor: string,
}
```
