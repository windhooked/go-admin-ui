<template>
  <el-form ref="basicInfoForm" :model="info" :rules="rules" label-width="150px">
    <el-row>
      <el-col :span="12">
        <el-form-item label="table name" prop="tableName">
          <el-input v-model="info.tableName" placeholder="Please enter the warehouse name" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="Table description" prop="tableComment">
          <el-input v-model="info.tableComment" placeholder="Please enter" />
        </el-form-item>
      </el-col>

      <el-col :span="12">
        <el-form-item label="Entity class name" prop="className">
          <el-input v-model="info.className" placeholder="Please enter" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="作者" prop="functionAuthor">
          <el-input v-model="info.functionAuthor" placeholder="Please enter" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item prop="isLogicalDelete">
          <span slot="label">
            Whether to logically delete
            <el-tooltip content="Currently only tombstones are supported" placement="top">
              <i class="el-icon-question" />
            </el-tooltip>
          </span>

          <el-radio-group v-model="info.isLogicalDelete">
            <el-radio label="1">是</el-radio>
            <el-radio label="0">否</el-radio>
          </el-radio-group>

        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item v-if="info.isLogicalDelete == '1'" label="logical deletion field" prop="logicalDeleteColumn">
          <el-input v-model="info.logicalDeleteColumn" placeholder="Please enter" />
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="Remarks" prop="remark">
          <el-input v-model="info.remark" type="textarea" :rows="3" />
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>
<script>
export default {
  name: 'BasicInfoForm',
  props: {
    info: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      rules: {
        tableName: [
          { required: true, message: 'Please enter the table name', trigger: 'blur' }
        ],
        tableComment: [
          { required: true, message: 'Please enter table description', trigger: 'blur' }
        ],
        className: [
          { required: true, message: 'Please enter the entity class name', trigger: 'blur' }
        ],
        functionAuthor: [
          { required: true, message: 'Please enter the author', trigger: 'blur' }
        ]
      }
    }
  }
}
</script>
