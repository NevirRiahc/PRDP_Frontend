<template>
   <el-form ref="userRef" :model="form" :rules="rules" label-width="116px">
      <el-form-item label="NickName" prop="nickName">
         <el-input v-model="form.nickName" maxlength="30" />
      </el-form-item>
      <el-form-item label="Phonenumber" prop="phonenumber">
         <el-input v-model="form.phonenumber" maxlength="11" />
      </el-form-item>
      <el-form-item label="Email" prop="email">
         <el-input v-model="form.email" maxlength="50" />
      </el-form-item>
      <el-form-item label="Sex">
         <el-radio-group v-model="form.sex">
            <el-radio value="0">male</el-radio>
            <el-radio value="1">female</el-radio>
         </el-radio-group>
      </el-form-item>
      <el-form-item>
      <el-button type="primary" @click="submit">Save</el-button>
      <el-button type="danger" @click="close">Close</el-button>
      </el-form-item>
   </el-form>
</template>

<script setup>
import { updateUserProfile } from "@/api/system/user";

const props = defineProps({
  user: {
    type: Object
  }
});

const { proxy } = getCurrentInstance();

const form = ref({});
const rules = ref({  
  nickName: [{ required: true, message: "Nickname cannot be empty", trigger: "blur" }],  
  email: [  
    { required: true, message: "Email address cannot be empty", trigger: "blur" },  
    { type: "email", message: "Please enter a valid email address", trigger: ["blur", "change"] }  
  ],  
  phonenumber: [  
    { required: true, message: "Phone number cannot be empty", trigger: "blur" },  
    { pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/, message: "Please enter a valid phone number", trigger: "blur" }  
  ],  
});

/** 提交按钮 */
function submit() {
  proxy.$refs.userRef.validate(valid => {
    if (valid) {
      updateUserProfile(form.value).then(response => {
        proxy.$modal.msgSuccess("Modified successfully");
        props.user.phonenumber = form.value.phonenumber;
        props.user.email = form.value.email;
      });
    }
  });
};

/** 关闭按钮 */
function close() {
  proxy.$tab.closePage();
};

// 回显当前登录用户信息
watch(() => props.user, user => {
  if (user) {
    form.value = { nickName: user.nickName, phonenumber: user.phonenumber, email: user.email, sex: user.sex };
  }
},{ immediate: true });
</script>
