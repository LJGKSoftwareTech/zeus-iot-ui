<!-- 属性表单 -->
<template>
  <el-form ref="dialogForm" :rules="rules" :model="formData" label-width="100px" class="attribute-form">
    <el-form-item label="属性名称" prop="attrName">
      <el-input v-model="formData.attrName" size="mini" :disabled="disabled"/>
    </el-form-item>
    <el-form-item label="标识符" prop="key">
      <el-input v-model="formData.key" size="mini" :disabled="disabled"/>
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />
        <span>可以包含数字、字母、-_.。更多特殊形式请见文档</span>
        <span v-if="false">标识符内可使用本产品 变量 的 键，且必须包含在{$}内；若变量的键为 PWD，则完整标识符可以为 xx{$PWD}xxx</span>
      </div>
    </el-form-item>
    <el-form-item label="来源类型" prop="source">
      <el-select v-model="formData.source" size="mini" placeholder="请选择来源类型" :disabled="disabled" @change="sourceChange">
        <el-option
          v-for="item in sourceList"
          :key="item.code"
          :label="item.name"
          :value="item.code"
        />
      </el-select>
    </el-form-item>
    <el-form-item v-if="formData.source === '18'" label="来源属性" prop="depAttrId">
      <el-select v-model="formData.depAttrId" size="mini" placeholder="请选择来源属性" :disabled="disabled">
        <el-option
          v-for="item in attrList"
          :key="item.attrId"
          :label="item.attrName"
          :value="item.attrId"
        />
      </el-select>
    </el-form-item>
    <el-form-item label="数据类型" prop="valueType">
      <el-select v-model="formData.valueType" size="mini" placeholder="请选择数据类型" :disabled="disabled">
        <el-option
          v-for="item in valueList"
          :key="item.code"
          :label="item.name"
          :value="item.code"
        />
      </el-select>
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />想要枚举？使用下面的值映射或数据预处理。
      </div>
    </el-form-item>
    <el-form-item v-if="formData.valueType == '3' || formData.valueType == '0'" label="单位">
      <el-select v-model="formData.units" filterable clearable size="mini" placeholder="请选择单位" :disabled="disabled">
        <el-option-group
          v-for="group in unitsList"
          :key="group.label"
          :label="group.label"
        >
          <el-option
            v-for="item in group.options"
            :key="item.code"
            :label="item.name"
            :value="item.code"
          />
        </el-option-group>
      </el-select>
    </el-form-item>
    <el-form-item v-if="formData.source === '0'" label="取数间隔" prop="delay">
      <el-input v-model.number="formData.delay" placeholder="请输入取数间隔" size="mini" class="input-with-select w500" @input="delayChange"></el-input>
      <el-select v-model="formData.unit" size="mini" placeholder="请选择" class="w100">
        <el-option label="秒" value="s"></el-option>
        <el-option label="分" value="m"></el-option>
        <el-option label="小时" value="h"></el-option>
      </el-select>
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />取数间隔在1秒到24小时之间。
      </div>
    </el-form-item>
    <el-form-item label="描述">
      <el-input v-model="formData.remark" type="textarea" rows="2" size="mini"/>
    </el-form-item>
    <el-form-item label="数据预处理">
      <Pretreatment ref="pretreatment" v-model="formData.processStepList" :disabled="disabled" />
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />若配置，则最终保存的是预处理后的数据。
      </div>
    </el-form-item>
    <el-form-item label="值映射">
      <el-select v-model="formData.valuemapid" clearable size="mini" placeholder="请选择值映射" :disabled="disabled">
        <el-option
          v-for="item in mapList"
          :key="item.valuemapid"
          :label="item.name"
          :value="item.valuemapid"
        />
      </el-select>
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />若配置，则实际保存的依然是原始值。只是方便让展现数据的可读性更好。
      </div>
    </el-form-item>
    <el-form-item label="标签">
      <Tag ref="tag" v-model="formData.tags" />
      <div class="el-form-item-tips">
        <svg-icon icon-class="tips" class="icon" />可在属性上增加自定义标签，以方便统计分析。
      </div>
    </el-form-item>
  </el-form>
</template>

<script>
import { getDictListByCode, groupDictByCode } from '@/api/system'
import { getAttrTrapperList, getDevValueMapList } from '@/api/deviceMgr'
import { getProductAttrTrapperList, getValueMapList } from '@/api/porductMgr'
import Pretreatment from '@/components/Detail/Pretreatment'
import Tag from '@/components/Detail/Tag'

export default {
  name: 'AttributeForm',
  components: {
    Pretreatment,
    Tag
  },
  props: {
    value: {
      type: Object,
      default() {
        return {}
      }
    },
    proId: {
      type: String,
      default: ''
    },
    isDev: Boolean
  },
  data() {
    const checkData = (rule, value, callback) => {
      if (value) {
        if (/[0-9a-zA-Z_-]/g.test(value)) {
          callback()
        } else {
          callback(new Error('请输入数字、字母或符号!'))
        }
      }
      callback()
    }
    return {
      formData: this.value,
      rules: {
        attrName: [
          { required: true, message: '请输入属性名称', trigger: 'blur' }
        ],
        delay: [
          { required: true, message: '请输入取数间隔', trigger: 'blur' }
        ],
        key: [
          { required: true, message: '请输入标识符', trigger: 'blur' },
          { validator: checkData, trigger: 'blur' }
        ],
        source: [
          { required: true, message: '请选择来源类型', trigger: 'change' }
        ],
        depAttrId: [
          { required: true, message: '请选择来源属性', trigger: 'change' }
        ],
        valueType: [
          { required: true, message: '请选择数据类型', trigger: 'change' }
        ]
      },
      sourceList: [],
      attrList: [],
      valueList: [],
      unitsList: [],
      mapList: [],
      prodId: null
    }
  },
  computed: {
    disabled() {
      return !!this.formData.attrId && !!this.formData.templateId
    }
  },
  watch: {
    formData: {
      deep: true,
      handler(val) {
        this.$emit('input', val)
      }
    }
  },
  created() {
    if (this.$route.query.id) {
      this.prodId = this.$route.query.id
      this.formData.productId = this.prodId
      this.getDict()
    }
  },
  methods: {
    getDict() {
      getDictListByCode({ dictTypeCode: 'ATTR_TYPE' }).then(res => {
        if (res.code == 200) {
          this.sourceList = res.data
        }
      })
      getDictListByCode({ dictTypeCode: 'DATA_TYPE' }).then(res => {
        if (res.code == 200) {
          this.valueList = res.data
        }
      })
      groupDictByCode({ dictTypeCode: 'UNITS' }).then(res => {
        if (res.code == 200) {
          this.unitsList = this.groupFormat(res.data)
        }
      })
      if (this.isDev) {
        getAttrTrapperList({ prodId: this.prodId }).then(res => {
          if (res.code == 200) {
            if (this.formData.attrId) {
              const arr = []
              res.data.forEach((i) => {
                if (i.attrId != this.formData.attrId) {
                  arr.push(i)
                }
              })
              this.attrList = arr
            } else {
              this.attrList = res.data
            }
          }
        })
        if (this.formData.templateId) {
          getValueMapList({ productId: this.proId }).then(res => {
            if (res.code == 200) {
              this.mapList = res.data
            }
          })
        } else {
          getDevValueMapList({ deviceId: this.prodId }).then(res => {
            if (res.code == 200) {
              this.mapList = res.data
            }
          })
        }
      } else {
        getProductAttrTrapperList({ prodId: this.prodId }).then(res => {
          if (res.code == 200) {
            if (this.formData.attrId) {
              const arr = []
              res.data.forEach((i) => {
                if (i.attrId != this.formData.attrId) {
                  arr.push(i)
                }
              })
              this.attrList = arr
            } else {
              this.attrList = res.data
            }
          }
        })
        getValueMapList({ productId: this.prodId }).then(res => {
          if (res.code == 200) {
            this.mapList = res.data
          }
        })
      }
    },
    delayChange() {
      if (this.formData.unit === '' || this.formData.unit === undefined) {
        this.$set(this.formData, 'unit', 's')
      }
    },
    groupFormat(data) {
      const list = []
      for (const key in data) {
        const item = {
          label: key,
          options: data[key]
        }
        list.push(item)
      }
      return list
    },
    validateForm() {
      let flag = false
      this.$refs.dialogForm.validate((valid) => {
        flag = valid
      })
      return flag && this.$refs.pretreatment.verification() && this.$refs.tag.verification()
    },
    sourceChange(val) {
      this.$refs.dialogForm.clearValidate('depAttrId')
      this.$refs.dialogForm.clearValidate('delay')
    }
  }
}
</script>

<style lang="scss" scoped>
.attribute-form {
  width: 600px;

  .w500{
    width: 500px;
  }

  .w100{
    width: 100px;
  }
}
</style>
