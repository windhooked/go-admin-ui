<template>
  <div class="app-container">
    <el-form :inline="true">
      <el-form-item label="Department name">
        <el-input
          v-model="queryParams.deptName"
          placeholder="Please enter the department name"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="status">
        <el-select v-model="queryParams.status" placeholder="Param Status" clearable size="small">
          <el-option
            v-for="dict in statusOptions"
            :key="dict.dictValue"
            :label="dict.dictLabel"
            :value="dict.dictValue"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          class="filter-item"
          type="primary"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery"
        >Search</el-button>
        <el-button
          v-permisaction="['system:sysdept:add']"
          class="filter-item"
          type="primary"
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
        >Add</el-button>
      </el-form-item>
    </el-form>

    <el-table
      v-loading="loading"
      :data="deptList"
      row-key="deptId"
      default-expand-all
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
    >
      <el-table-column prop="deptName" label="部门名称" />
      <el-table-column prop="sort" label="排序" width="200" />
      <el-table-column prop="status" label="状态" :formatter="statusFormat" width="100">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.status === '1' ? 'danger' : 'success'"
            disable-transitions
          >{{ statusFormat(scope.row) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center" prop="createdAt" width="200">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createdAt) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            v-permisaction="['system:sysdept:edit']"
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
          >Edit</el-button>
          <el-button
            v-permisaction="['system:sysdept:add']"
            size="mini"
            type="text"
            icon="el-icon-plus"
            @click="handleAdd(scope.row)"
          >Add</el-button>
          <el-button
            v-if="scope.row.p_id != 0"
            v-permisaction="['system:sysdept:remove']"
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
          >Delete</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 添加或修改部门对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="600px">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-row>
          <el-col :span="24">
            <el-form-item label="superior department" prop="parentId">
              <treeselect
                v-model="form.parentId"
                :options="deptOptions"
                :normalizer="normalizer"
                :show-count="true"
                placeholder="Select superior department"
                :is-disabled="isEdit"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Department name" prop="deptName">
              <el-input v-model="form.deptName" placeholder="Please enter the department name" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="display order" prop="orderNum">
              <el-input-number v-model="form.sort" controls-position="right" :min="0" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="person in charge" prop="leader">
              <el-input v-model="form.leader" placeholder="Please enter the person in charge" maxlength="20" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="contact phone" prop="phone">
              <el-input v-model="form.phone" placeholder="Please enter the contact number" maxlength="11" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="mailbox" prop="email">
              <el-input v-model="form.email" placeholder="Please enter your mailbox" maxlength="50" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Department Status">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in statusOptions"
                  :key="dict.dictValue"
                  :label="dict.dictValue"
                >{{ dict.dictLabel }}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">Submit</el-button>
        <el-button @click="cancel">Cancel</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getDeptList, getDept, delDept, addDept, updateDept } from '@/api/system/dept'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

export default {
  name: 'Dept',
  components: { Treeselect },
  data() {
    return {
      // 遮罩层
      loading: true,
      // 表格树数据
      deptList: [],
      // 部门树选项
      deptOptions: [],
      // 弹出层标题
      title: '',
      isEdit: false,
      // 是否显示弹出层
      open: false,
      // 状态数据字典
      statusOptions: [],
      // 查询参数
      queryParams: {
        deptName: undefined,
        status: undefined
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        parentId: [
          { required: true, message: 'The superior department cannot be empty', trigger: 'blur' }
        ],
        deptName: [
          { required: true, message: 'Department name cannot be empty', trigger: 'blur' }
        ],
        sort: [
          { required: true, message: 'The menu order cannot be empty', trigger: 'blur' }
        ],
        email: [
          {
            type: 'email',
            message: "'Please enter the correct email address",
            trigger: ['blur', 'change']
          }
        ],
        phone: [
          {
            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
            message: 'Please enter the correct mobile phone number',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getList()
    this.getDicts('sys_normal_disable').then(response => {
      this.statusOptions = response.data
    })
  },
  methods: {
    /** Query department list */
    getList() {
      this.loading = true
      getDeptList(this.queryParams).then(response => {
        this.deptList = response.data
        this.loading = false
      })
    },
    /** Convert department data structure */
    normalizer(node) {
      if (node.children && !node.children.length) {
        delete node.children
      }
      return {
        id: node.deptId,
        label: node.deptName,
        children: node.children
      }
    },
    /** Query department drop-down tree structure */
    getTreeselect(e) {
      getDeptList().then(response => {
        this.deptOptions = []

        if (e === 'update') {
          const dept = { deptId: 0, deptName: 'depname', children: [], isDisabled: true }
          dept.children = response.data
          this.deptOptions.push(dept)
        } else {
          const dept = { deptId: 0, deptName: 'depName', children: [] }
          dept.children = response.data
          this.deptOptions.push(dept)
        }
      })
    },
    // Dictionary status dictionary translation
    statusFormat(row) {
      return this.selectDictLabel(this.statusOptions, row.status)
    },
    // cancel button
    cancel() {
      this.open = false
      this.reset()
    },
    // Form reset
    reset() {
      this.form = {
        deptId: undefined,
        parentId: undefined,
        deptName: undefined,
        sorc: undefined,
        leader: undefined,
        phone: undefined,
        email: undefined,
        status: '0'
      }
    },
    /** Search button operation */
    handleQuery() {
      this.getList()
    },
    /** Add button operation */
    handleAdd(row) {
      this.reset()
      this.getTreeselect('add')
      if (row !== undefined) {
        this.form.parentId = row.deptId
      }
      this.open = true
      this.title = 'Add Department'
      this.isEdit = false
    },
    /** Modify button operation */
    handleUpdate(row) {
      this.reset()
      this.getTreeselect('update')

      getDept(row.deptId).then(response => {
        this.form = response.data
        this.open = true
        this.title = 'Modify Department'
        this.isEdit = true
      })
    },
    /** Submit button */
    submitForm: function() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          if (this.form.deptId !== undefined) {
            updateDept(this.form).then(response => {
              if (response.code === 200) {
                this.msgSuccess('Modified successfully')
                this.open = false
                this.getList()
              } else {
                this.msgError(response.msg)
              }
            })
          } else {
            addDept(this.form).then(response => {
              if (response.code === 200) {
                this.msgSuccess('Successfully added')
                this.open = false
                this.getList()
              } else {
                this.msgError(response.msg)
              }
            })
          }
        }
      })
    },
    /** Delete button operation */
    handleDelete(row) {
      this.$confirm(
        'Are you sure to delete the data item with the name "' + row.deptName + '"?',
        'caveat',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }
      )
        .then(function() {
          return delDept(row.deptId)
        })
        .then(() => {
          this.getList()
          this.msgSuccess('Successfully deleted')
        })
        .catch(function() {})
    }
  }
}
</script>
