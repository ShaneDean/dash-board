<template>
  <div class="db-padding-setting">
    <div class="padding-box">
      <el-form-item
        label="上边距"
        label-width="120px"
      >
        <el-input-number
          v-model.number="paddingValue[0]"
          class="input-top input-item"
          size="mini"
          :min="0"
          :step="1"
          @change="paddingChange"
        />
      </el-form-item>
      <el-form-item
        label="下边距"
        label-width="120px"
      >
        <el-input-number
          v-model.number="paddingValue[2]"
          class="input-bottom input-item"
          size="mini"
          :min="0"
          :step="1"
          @change="paddingChange"
        />
      </el-form-item>
      <el-form-item
        label="左边距"
        label-width="120px"
      >
        <el-input-number
          v-model.number="paddingValue[3]"
          class="input-left input-item db-el-input"
          size="mini"
          :min="0"
          :step="1"
          @change="paddingChange"
        />
      </el-form-item>
      <el-form-item
        label="右边距"
        label-width="120px"
      >
        <el-input-number
          v-model.number="paddingValue[1]"
          class="input-right input-item"
          size="mini"
          :min="0"
          :step="1"
          @change="paddingChange"
        />
      </el-form-item>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PaddingSetting',
  model: {
    prop: 'padding',
    event: 'input'
  },
  props: {
    padding: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      position: 0
    }
  },
  computed: {
    paddingValue () {
      return this.padding
    }
  },
  methods: {
    paddingChange () {
      this.$emit('input', this.paddingValue)
    }
  }
}
</script>

<style lang="scss" scoped>
.db-padding-setting{
  .padding-box{
    width: 160px;
    height: 190px;
    position: relative;
  }
  .padding-center{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    width: 30%;
    height: 30%;
    background-color: var(--db-background-1);
  }
}
</style>
