<template>
  <el-form ref="form" :model="user" :rules="rules" label-width="80px">
    <el-form-item label="旧密码" prop="oldPassword">
      <el-input
        v-model="user.oldPassword"
        placeholder="Please enter the old password"
        type="password"
      />
    </el-form-item>
    <el-form-item label="新密码" prop="newPassword">
      <el-input
        v-model="user.newPassword"
        placeholder="Please enter a new password"
        type="password"
      />
    </el-form-item>
    <el-form-item label="确认密码" prop="confirmPassword">
      <el-input
        v-model="user.confirmPassword"
        placeholder="Please confirm password"
        type="password"
      />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" size="mini" @click="submit">submit</el-button>
      <el-button type="danger" size="mini" @click="close">close</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { updateUserPwd } from '@/api/system/sysuser'

export default {
  data() {
    const equalToPassword = (rule, value, callback) => {
      if (this.user.newPassword !== value) {
        callback(new Error('The passwords entered twice are inconsistent'))
      } else {
        callback()
      }
    }
    return {
      test: '1test',
      user: {
        oldPassword: undefined,
        newPassword: undefined,
        confirmPassword: undefined
      },
      // 表单校验
      rules: {
        oldPassword: [
          {
            required: true,
            message: 'Old password cannot be empty',
            trigger: 'blur'
          }
        ],
        newPassword: [
          {
            required: true,
            message: 'The new password cannot be empty',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 20,
            message: 'Length between 6 and 20 characters',
            trigger: 'blur'
          }
        ],
        confirmPassword: [
          {
            required: true,
            message: 'Confirm password cannot be empty',
            trigger: 'blur'
          },
          { required: true, validator: equalToPassword, trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    submit() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          updateUserPwd(this.user.oldPassword, this.user.newPassword).then(
            response => {
              if (response.code === 200) {
                this.msgSuccess('Modified successfully')
              } else {
                this.msgError(response.msg)
              }
            }
          )
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
