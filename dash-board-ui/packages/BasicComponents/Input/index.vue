<template>
  <div class="db-design-wrap">
    <div class="basic-component-input">
      <el-input
        :id="`el-input-${config.code}`"
        v-model="config.customize.value"
        type="text"
        resize="both"
        class="input"
        :placeholder="config.customize.placeholderStyle.placeholder"
        :style="{ backgroundColor: config.customize.backgroundStyle.backgroundColor }"
      >
        <i
          v-if="config.customize.icon.position === 'left' && config.customize.icon.name"
          slot="prefix"
          class="el-input__icon"
          :class="config.customize.icon.position === 'left' ? config.customize.icon.name : ''"
        />
        <i
          v-if="config.customize.icon.position === 'right' && config.customize.icon.name"
          slot="suffix"
          class="el-input__icon"
          :class="config.customize.icon.position === 'right' ? config.customize.icon.name : ''"
        />
      </el-input>
    </div>
  </div>
</template>

<script>
import commonMixins from 'dashPackages/js/mixins/commonMixins'
import linkageMixins from 'dashPackages/js/mixins/linkageMixins'
export default {
  name: 'BasicComponentInput',
  mixins: [commonMixins, linkageMixins],
  props: {
    // 卡片的属性
    config: {
      type: Object,
      default: () => ({})
    }
  },
  data () {
    return { }
  },
  mounted () {
    this.updateComponent()
    this.chartInit()
  },
  methods: {
    updateComponent () {
      const input = document.querySelector(`#el-input-${this.config.code}`)

      // const inputIcon = input.querySelector(`.${this.config.customize.icon.name}`)
      input.style.backgroundColor = this.config.customize.backgroundStyle.backgroundColor
      input.style.fontSize = this.config.customize.inputStyle.fontSize + 'px'
      input.style.color = this.config.customize.inputStyle.color
      input.style.borderColor = this.config.customize.borderStyle.borderColor
      input.style.borderWidth = this.config.customize.borderStyle.borderWidth + 'px'
      input.style.borderStyle = this.config.customize.borderStyle.borderStyle
      input.style.borderRadius = this.config.customize.borderStyle.borderRadius + 'px'
      // inputIcon.style.fontSize = this.config.customize.inputStyle.fontSize + 'px'
      if (this.config.customize.icon.name) {
        const inputIcon = document.querySelector(`.${this.config.customize.icon.name}`)
        inputIcon.style.fontSize = this.config.customize.inputStyle.fontSize + 'px'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  @import '../../assets/style/chartStyle.scss';
.db-design-wrap {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  .basic-component-input{
    width: 80%;
  }
  .title-left {
    display: block;
    white-space: nowrap;
    align-self: center;
  }
  // .title-top{
  //   display: block;
  //   text-align: center;
  //   white-space: nowrap;
  // }

  .el-input {
    height: 100%;
    width: 100%;

    ::v-deep .el-input__inner {
      height: 100%;
      width: 100%;
    }
  }

}
</style>
