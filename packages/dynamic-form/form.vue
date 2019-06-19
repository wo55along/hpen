<template>
  <div class="dynamic-form" :style="style">
    <el-form
      v-if="_value"
      ref="dynamic-form"
      :model="_value"
      :rules="descriptors">
      <dynamic-form-item
        v-for="(descriptor, key) in descriptors"
        v-model="_value[key]"
        :key="key"
        :lang="lang"
        :label="findTypeDescriptor(descriptor).label || key"
        :prop="key"
        :label-width="labelWidth"
        :descriptor="descriptor"
        :size="size"
        :background-color="backgroundColor"
        :bg-color-offset="bgColorOffset">
      </dynamic-form-item>
      <el-form-item v-if="$slots.operations" class="operations" :label-width="labelWidth">
        <slot name="operations"></slot>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { Form, FormItem } from 'element-ui'
import DynamicFormItem from '../dynamic-form-item/form-item'
import { isComplexType, getLabelWidth, findTypeDescriptor } from '../utils'

export default {
  name: 'dynamic-form',
  props: {
    value: {
      type: Object,
      required: true
    },
    lang: {
      type: String,
      default: 'en_US',
      validator: val => ['en_US', 'zh_CN'].includes(val)
    },
    /**
     * descriptor of value, extend from https://github.com/yiminghe/async-validator
     */
    descriptors: {
      type: Object,
      required: true
    },
    /**
     * size of the input component
     */
    size: {
      type: String,
      default: 'small'
    },
    /**
     * background-color of form
     */
    backgroundColor: {
      type: String,
      default: '#FFFFFF'
    },
    /**
     * font-size of form
     */
    fontSize: {
      type: Number,
      default: 14
    },
    /**
     * darken sub-form's background-color with offset while get positive integer
     */
    bgColorOffset: {
      type: Number,
      default: 8
    }
  },
  components: {
    ElForm: Form,
    ElFormItem: FormItem,
    DynamicFormItem
  },
  computed: {
    _value: {
      get () {
        return this.value
      },
      set (value) {
        this.$emit('input', value)
      }
    },
    labelWidth () {
      return getLabelWidth(this.descriptors, this.fontSize)
    },
    style () {
      const style = {
        fontSize: `${this.fontSize}px`,
        backgroundColor: this.backgroundColor
      }
      return style
    }
  },
  data () {
    return {}
  },
  created () {
    this.init()
  },
  methods: {
    findTypeDescriptor,
    init () {
      this.initValue()
    },
    initValue () {
      for (const key in this.descriptors) {
        this.setValueKey(this, this._value, key, this.descriptors[key])
      }
    },
    setValueKey (target, value, key, descriptor) {
      if (isComplexType(descriptor.type)) {
        if (descriptor.type === 'object') {
          // object
          if (descriptor.fields) {
            // normal object
            if (value[key] === undefined) {
              target.$set(value, key, {})
            }
            for (const _key in descriptor.fields) {
              target.setValueKey(target, value[key], _key, descriptor.fields[_key])
            }
          } else {
            // hashmap
            if (value[key] === undefined) {
              target.$set(value, key, {})
            }
          }
        } else {
          // array
          if (value[key] === undefined) {
            target.$set(value, key, [])
          }
        }
      } else {
        if (value[key] === undefined) {
          target.$set(value, key, null)
        }
      }
    },
    validate (func) {
      if (typeof func === 'function') {
        this.$refs['dynamic-form'].validate(valid => {
          func(valid)
        })
      } else {
        return new Promise((resolve, reject) => {
          this.$refs['dynamic-form'].validate(valid => {
            resolve(valid)
          })
        })
      }
    },
    resetFields () {
      this.$refs['dynamic-form'].resetFields()
    },
    clearValidate () {
      this.$refs['dynamic-form'].clearValidate()
    }
  }
}
</script>

<style lang="scss" scoped>
</style>