<template>
  <div class="demo-page">
    <h1>el-input maxlength 场景示例</h1>

    <section class="demo-section">
      <h2>包含 maxlength</h2>

      <el-input v-model="form.username" maxlength="20" placeholder="固定 maxlength" />

      <el-input
        v-model="form.nickname"
        :maxlength="10"
        show-word-limit
        placeholder="动态绑定为数字"
      />

      <el-input
        v-model="form.email"
        :maxlength="maxEmailLength"
        placeholder="绑定变量 maxlength"
      />

      <el-input
        v-model="form.remark"
        type="textarea"
        maxlength="100"
        show-word-limit
        placeholder="textarea + maxlength"
      />

      <el-input
        v-if="enableLimitedInput"
        v-model="form.conditionalLimited"
        maxlength="30"
        placeholder="条件渲染且包含 maxlength"
      />
    </section>

    <section class="demo-section">
      <h2>不包含 maxlength</h2>

      <el-input v-model="form.address" placeholder="未设置 maxlength" />

      <el-input
        v-model="form.phone"
        clearable
        placeholder="带其他属性但不含 maxlength"
      />

      <el-input
        v-model="form.description"
        type="textarea"
        :rows="4"
        placeholder="textarea 不含 maxlength"
      />

      <el-input v-model="form.nonSelfClose"></el-input>

      <el-input
        v-if="enableUnlimitedInput"
        v-model="form.conditionalUnlimited"
        placeholder="条件渲染但不含 maxlength"
      />

      <el-input
        v-model="form.boundUndefined"
        :maxlength="undefined"
        placeholder="绑定了 undefined"
      />
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const maxEmailLength = ref(50)
const enableLimitedInput = ref(true)
const enableUnlimitedInput = ref(true)

const form = ref({
  username: '',
  nickname: '',
  email: '',
  remark: '',
  conditionalLimited: '',
  address: '',
    phone: '',
    description: '',
    nonSelfClose: '',
    conditionalUnlimited: '',
    boundUndefined: '',
  })
</script>

<style scoped>
.demo-page {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px;
}

.demo-section {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 32px;
}
</style>
