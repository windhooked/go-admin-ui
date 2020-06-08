<template>
  <div class="app-container">
    <el-row :gutter="20">
      <!--Department Data-->
      <el-col :span="4" :xs="24">
        <div class="head-container">
          <el-input
            v-model="deptName"
            placeholder="Please enter the department name"
            clearable
            size="small"
            prefix-icon="el-icon-search"
            style="margin-bottom: 20px"
          />
        </div>
        <div class="head-container">
          <el-tree
            ref="tree"
            :data="deptOptions"
            :props="defaultProps"
            :expand-on-click-node="false"
            :filter-node-method="filterNode"
            default-expand-all
            @node-click="handleNodeClick"
          />
        </div>
      </el-col>
      <!--User data-->
      <el-col :span="20" :xs="24">
        <el-form ref="queryForm" :model="queryParams" :inline="true" label-width="68px">
          <el-form-item label="user name" prop="username">
            <el-input
              v-model="queryParams.username"
              placeholder="Please enter a user name"
              clearable
              size="small"
              style="width: 240px"
              @keyup.enter.native="handleQuery"
            />
          </el-form-item>
          <el-form-item label="Mobile number" prop="phone">
            <el-input
              v-model="queryParams.phone"
              placeholder="Please enter your mobile number"
              clearable
              size="small"
              style="width: 240px"
              @keyup.enter.native="handleQuery"
            />
          </el-form-item>
          <el-form-item label="status" prop="status">
            <el-select
              v-model="queryParams.status"
              placeholder="User status"
              clearable
              size="small"
              style="width: 240px"
            >
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
              type="primary"
              icon="el-icon-search"
              size="mini"
              @click="handleQuery"
            >handle Query</el-button>
            <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">reset Query</el-button>
          </el-form-item>
        </el-form>

        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button
              v-permisaction="['system:sysuser:add']"
              type="primary"
              icon="el-icon-plus"
              size="mini"
              @click="handleAdd"
            >Add</el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-permisaction="['system:sysuser:edit']"
              type="success"
              icon="el-icon-edit"
              size="mini"
              :disabled="single"
              @click="handleUpdate"
            >Edit</el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-permisaction="['system:sysuser:remove']"
              type="danger"
              icon="el-icon-delete"
              size="mini"
              :disabled="multiple"
              @click="handleDelete"
            >Delete</el-button>
          </el-col>
        </el-row>

        <el-table v-loading="loading" :data="userList" @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="45" align="center" />
          <el-table-column label="UserId" width="50" align="center" prop="userId" />
          <el-table-column
            label="Username"
            align="center"
            prop="username"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            label="NickName"
            align="center"
            prop="nickName"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            label="DepartmentName"
            align="center"
            prop="deptName"
            :show-overflow-tooltip="true"
          />
          <el-table-column label="Phone" align="center" prop="phone" width="120" />
          <el-table-column label="Fixme" width="50" align="center">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.status"
                active-value="0"
                inactive-value="1"
                @change="handleStatusChange(scope.row)"
              />
            </template>
          </el-table-column>
          <el-table-column label="Created time" align="center" prop="createdAt" width="165">
            <template slot-scope="scope">
              <span>{{ parseTime(scope.row.createdAt) }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="Action"
            align="center"
            width="220"
            class-name="small-padding fixed-width"
          >
            <template slot-scope="scope">
              <el-button
                v-permisaction="['system:sysuser:edit']"
                size="mini"
                type="text"
                icon="el-icon-edit"
                @click="handleUpdate(scope.row)"
              >Edit</el-button>
              <el-button
                v-if="scope.row.userId !== 1"
                v-permisaction="['system:sysuser:remove']"
                size="mini"
                type="text"
                icon="el-icon-delete"
                @click="handleDelete(scope.row)"
              >Delete</el-button>
              <el-button
                v-permisaction="['system:sysuser:resetPassword']"
                size="mini"
                type="text"
                icon="el-icon-key"
                @click="handleResetPwd(scope.row)"
              >Reset</el-button>
            </template>
          </el-table-column>
        </el-table>

        <pagination
          v-show="total>0"
          :total="total"
          :page.sync="queryParams.pageIndex"
          :limit.sync="queryParams.pageSize"
          @pagination="getList"
        />
      </el-col>
    </el-row>

    <!-- Add or modify parameter configuration dialog -->
    <el-dialog :title="title" :visible.sync="open" width="600px">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="user nickname" prop="nickName">
              <el-input v-model="form.nickName" placeholder="Please enter user nickname" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Owned Department" prop="deptId">
              <treeselect
                v-model="form.deptId"
                :options="deptOptions"
                :normalizer="normalizer"
                placeholder="Please select attribution department"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Mobile number" prop="phone">
              <el-input
                v-model="form.phone"
                placeholder="Please enter the phone number"
                maxlength="11"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="mailbox" prop="email">
              <el-input
                v-model="form.email"
                placeholder="Please enter your mailbox"
                maxlength="50"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="user name" prop="username">
              <el-input v-model="form.username" placeholder="Please enter user name" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item v-if="form.userId == undefined" label="user password" prop="password">
              <el-input
                v-model="form.password"
                placeholder="Please enter user password"
                type="password"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="user gender">
              <el-select v-model="form.sex" placeholder="Please select">
                <el-option
                  v-for="dict in sexOptions"
                  :key="dict.dictValue"
                  :label="dict.dictLabel"
                  :value="dict.dictValue"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="status">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in statusOptions"
                  :key="dict.dictValue"
                  :label="dict.dictValue"
                >{{ dict.dictLabel }}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>

          <el-col :span="12">
            <el-form-item label="post">
              <el-select v-model="form.postId" placeholder="Please select" @change="$forceUpdate()">
                <el-option
                  v-for="item in postOptions"
                  :key="item.postId"
                  :label="item.postName"
                  :value="item.postId"
                  :disabled="item.status == 1"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="role">
              <el-select v-model="form.roleId" placeholder="Please select" @change="$forceUpdate()">
                <el-option
                  v-for="item in roleOptions"
                  :key="item.roleId"
                  :label="item.roleName"
                  :value="item.roleId"
                  :disabled="item.status == 1"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Remarks">
              <el-input v-model="form.remark" type="textarea" placeholder="Please enter content" />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">Submit</el-button>
        <el-button @click="cancel">Cancel</el-button>
      </div>
    </el-dialog>

    <!-- User import dialog -->
    <el-dialog :title="upload.title" :visible.sync="upload.open" width="400px">
      <el-upload
        ref="upload"
        :limit="1"
        accept=".xlsx, .xls"
        :headers="upload.headers"
        :action="upload.url + '?updateSupport=' + upload.updateSupport"
        :disabled="upload.isUploading"
        :on-progress="handleFileUploadProgress"
        :on-success="handleFileSuccess"
        :auto-upload="false"
        drag
      >
        <i class="el-icon-upload" />
        <div class="el-upload__text">
          Drag the file here, or
          <em>Click to upload</em>
        </div>
        <div slot="tip" class="el-upload__tip">
          <el-checkbox v-model="upload.updateSupport" />Whether to update existing user data
          <el-link type="info" style="font-size:12px" @click="importTemplate">Download template</el-link>
        </div>
        <div
          slot="tip"
          class="el-upload__tip"
          style="color:red"
        >Tip: Only import "xls" or "xlsx" format files!</div>
      </el-upload>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFileForm">Submit file form</el-button>
        <el-button @click="upload.open = false">Upload Open</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listUser,
  getUser,
  delUser,
  addUser,
  updateUser,
  exportUser,
  resetUserPwd,
  changeUserStatus,
  importTemplate,
  getUserInit
} from '@/api/system/sysuser'
import { getToken } from '@/utils/auth'
import { treeselect } from '@/api/system/dept'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

export default {
  name: 'User',
  components: { Treeselect },
  data() {
    return {
      // 遮罩层
      // Mask layer
      loading: true,
      // 选中数组
      // select the array
      ids: [],
      // 非单个禁用
      // Not single disabled
      single: true,
      // 非多个禁用
      // Not multiple disabled
      multiple: true,
      // 总条数
      // Total number
      total: 0,
      // 用户表格数据
      // User form data
      userList: null,
      // 弹出层标题
      // Popup layer title
      title: '',
      // 部门树选项
      // Department tree option
      deptOptions: undefined,
      // 是否显示弹出层
      // Whether to display the pop-up layer
      open: false,
      // 部门名称
      // Department name
      deptName: undefined,
      // 默认密码
      // default password
      initPassword: undefined,
      // 日期范围
      // Date range
      dateRange: [],
      // 状态数据字典
      // State data dictionary
      statusOptions: [],
      // 性别状态字典
      // Gender status dictionary
      sexOptions: [],
      // 岗位选项
      // Job options
      postOptions: [],
      // 角色选项
      // Role options
      roleOptions: [],
      // 表单参数
      // Form parameters
      form: {},
      defaultProps: {
        children: 'children',
        label: 'deptName'
      },
      // 用户导入参数
      // User import parameters
      upload: {
        // 是否显示弹出层（用户导入）
        // Whether to display the pop-up layer (user import)
        open: false,
        // 弹出层标题（用户导入）
        // Popup layer title (user import)
        title: '',
        // 是否禁用上传
        // Whether to disable upload
        isUploading: false,
        // 是否更新已经存在的用户数据
        // Whether to update the existing user data
        updateSupport: 0,
        // 设置上传的请求头部
        // Set the upload request header
        headers: { Authorization: 'Bearer ' + getToken() },
        // 上传的地址
        // Uploaded address
        url: process.env.VUE_APP_BASE_API + '/system/user/importData'
      },
      // 查询参数
      // Query parameters
      queryParams: {
        pageIndex: 1,
        pageSize: 10,
        username: undefined,
        phone: undefined,
        status: undefined,
        deptId: undefined
      },
      // 表单校验
      // Form verification
      rules: {
        username: [
          {
            required: true,
            message: 'User name cannot be empty',
            trigger: 'blur'
          }
        ],
        nickName: [
          {
            required: true,
            message: 'User nickname cannot be empty',
            trigger: 'blur'
          }
        ],
        deptId: [
          {
            required: true,
            message: 'The attribution department cannot be empty',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: 'User password cannot be empty',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: 'Email address cannot be empty',
            trigger: 'blur'
          },
          {
            type: 'email',
            message: "'Please enter the correct email address",
            trigger: ['blur', 'change']
          }
        ],
        phone: [
          {
            required: true,
            message: 'Mobile phone number cannot be empty',
            trigger: 'blur'
          },
          {
            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
            message: 'Please enter the correct mobile phone number',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  watch: {
    // Filter department tree based on name
    deptName(val) {
      this.$refs.tree.filter(val)
    }
  },
  created() {
    this.getList()
    this.getTreeselect()
    this.getDicts('sys_normal_disable').then(response => {
      this.statusOptions = response.data
    })
    this.getDicts('sys_user_sex').then(response => {
      this.sexOptions = response.data
    })
    this.getConfigKey('sys.user.initPassword').then(response => {
      this.initPassword = response.msg
    })
  },
  methods: {
    /** Query user list */
    getList() {
      this.loading = true
      listUser(this.addDateRange(this.queryParams, this.dateRange)).then(
        response => {
          this.userList = response.data.list
          this.total = response.data.count
          this.loading = false
        }
      )
    },
    /** Query department drop-down tree structure */
    getTreeselect() {
      treeselect().then(response => {
        this.deptOptions = response.data
      })
    },
    // filter nodes
    filterNode(value, data) {
      if (!value) return true
      return data.deptName.indexOf(value) !== -1
    },
    // Node click event
    handleNodeClick(data) {
      this.queryParams.deptId = data.deptId
      this.getList()
    },
    /** Convert menu data structure */
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
    // User status modification
    handleStatusChange(row) {
      const text = row.status === '0' ? 'Enable' : 'Disable'
      this.$confirm(
        'Confirm to "' + text + '""' + row.username + '"User?"',
        'caveat',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }
      )
        .then(function() {
          return changeUserStatus(row.userId, row.status)
        })
        .then(() => {
          this.msgSuccess(text + 'success')
        })
        .catch(function() {
          row.status = row.status === '0' ? '1' : '0'
        })
    },
    // cancel button
    cancel() {
      this.open = false
      this.reset()
    },
    // Form reset
    reset() {
      this.form = {
        userId: undefined,
        deptId: undefined,
        username: undefined,
        nickName: undefined,
        password: undefined,
        phone: undefined,
        email: undefined,
        sex: undefined,
        status: '0',
        remark: undefined,
        postIds: undefined,
        roleIds: undefined
      }
      this.resetForm('form')
    },
    /** Search button operation */
    handleQuery() {
      this.queryParams.page = 1
      this.getList()
    },
    /** Reset button operation */
    resetQuery() {
      this.dateRange = []
      this.resetForm('queryForm')
      this.handleQuery()
    },
    // check the data in the checkbox
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.userId)
      this.single = selection.length !== 1
      this.multiple = !selection.length
    },
    /** Add button operation */
    handleAdd() {
      this.reset()
      this.getTreeselect()
      getUserInit().then(response => {
        this.postOptions = response.data.posts
        this.roleOptions = response.data.roles
        this.open = true
        this.title = 'Add User'
        this.form.password = this.initPassword
      })
    },
    /** Modify button operation */
    handleUpdate(row) {
      this.reset()
      this.getTreeselect()

      const userId = row.userId || this.ids
      getUser(userId).then(response => {
        this.form = response.data
        this.postOptions = response.posts
        this.roleOptions = response.roles
        this.form.postIds = response.postIds[0]
        this.form.roleIds = response.roleIds[0]
        this.open = true
        this.title = 'Modify User'
        this.form.password = ''
      })
    },
    /** Reset password button operation */
    handleResetPwd(row) {
      this.$prompt(
        'Please enter "' + row.username + '"new password',
        'prompt',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel'
        }
      )
        .then(({ value }) => {
          resetUserPwd(row.userId, value).then(response => {
            if (response.code === 200) {
              this.msgSuccess(
                'Successfully modified, the new password is:' + value
              )
            } else {
              this.msgError(response.msg)
            }
          })
        })
        .catch(() => {})
    },
    /** Submit button */
    submitForm: function() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          if (this.form.userId !== undefined) {
            updateUser(this.form).then(response => {
              if (response.code === 200) {
                this.msgSuccess('Successfully modified')
                this.open = false
                this.getList()
              } else {
                this.msgError(response.msg)
              }
            })
          } else {
            addUser(this.form).then(response => {
              if (response.code === 200) {
                this.msgSuccess('Add success')
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
      const userIds = row.userId || this.ids
      this.$confirm(
        'Are you sure to delete the data item with user ID "' + userIds + '"?',
        'caveat',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }
      )
        .then(function() {
          return delUser(userIds)
        })
        .then(() => {
          this.getList()
          this.msgSuccess('Successfully deleted')
        })
        .catch(function() {})
    },
    /** Export button operation */
    handleExport() {
      const queryParams = this.queryParams
      this.$confirm('Are you sure to export all user data items?', 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      })
        .then(function() {
          return exportUser(queryParams)
        })
        .then(response => {
          this.download(response.msg)
        })
        .catch(function() {})
    },
    /** Import button operation */
    handleImport() {
      this.upload.title = 'User Import'
      this.upload.open = true
    },
    /** Download template operation */
    importTemplate() {
      importTemplate().then(response => {
        this.download(response.msg)
      })
    },
    // Process during file upload
    handleFileUploadProgress(event, file, fileList) {
      this.upload.isUploading = true
    },
    // File upload successfully processed
    handleFileSuccess(response, file, fileList) {
      this.upload.open = false
      this.upload.isUploading = false
      this.$refs.upload.clearFiles()
      this.$alert(response.msg, 'Import result', {
        dangerouslyUseHTMLString: true
      })
      this.getList()
    },
    // Submit upload file
    submitFileForm() {
      this.$refs.upload.submit()
    }
  }
}
</script>
