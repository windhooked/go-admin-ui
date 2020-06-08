<template>
  <el-form ref="form" :model="user" :rules="rules" label-width="80px">
    <el-form-item label="user nickname" prop="nickName">
      <el-input v-model="user.nickName" />
    </el-form-item>
    <el-form-item label="Mobile number" prop="phone">
      <el-input v-model="user.phone" maxlength="11" />
    </el-form-item>
    <el-form-item label="mailbox" prop="email">
      <el-input v-model="user.email" maxlength="50" />
    </el-form-item>
    <el-form-item label="gender">
      <el-radio-group v-model="user.sex">
        <el-radio label="0"> male </el-radio>
        <el-radio label="1"> female</el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" size="mini" @click="submit">submit</el-button>
      <el-button type="danger" size="mini" @click="close">close</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { updateUser } from '@/api/system/sysuser'

export default {
  props: {
    // eslint-disable-next-line vue/require-default-prop
    user: { type: Object }
  },
  data() {
    return {
      // 表单校验
      rules: {
        nickName: [
          { required: true, message: 'User nickname cannot be empty', trigger: 'blur' }
        ],
        email: [
          { required: true, message: 'Email address cannot be empty', trigger: 'blur' },
          {
            type: 'email',
            message: "'Please enter the correct email address",
            trigger: ['blur', 'change']
          }
        ],
        phone: [
          { required: true, message: 'Mobile number cannot be empty', trigger: 'blur' },
          {
            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
            message: 'Please enter the correct mobile phone number',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    submit() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          updateUser(this.user).then(response => {
            if (response.code === 200) {
              this.msgSuccess('Modified successfully')
            } else {
              this.msgError(response.msg)
            }
          })
        }
      })
    },
    close() {
      this.$store.dispatch('tagsView/delView', this.$route)
      this.$router.push({ path: '/index' })
    }
  }
}
</script>
