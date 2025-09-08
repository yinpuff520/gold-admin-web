<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">用户ID</label>
        <el-input v-model="query.userId" clearable placeholder="用户ID" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">用户名</label>
        <el-input v-model="query.username" clearable placeholder="用户名" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">用户类型</label>
        <el-input v-model="query.userType" clearable placeholder="用户类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">币种名</label>
        <el-input v-model="query.coin" clearable placeholder="币种名" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">账户类型</label>
        <el-input v-model="query.accountType" clearable placeholder="账户类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.updateTime"
          start-placeholder="updateTimeStart"
          end-placeholder="updateTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="id">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户ID">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户名">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户类型">
            <el-select v-model="form.userType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.user_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="用户备注">
            <el-input v-model="form.remark" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="钱包余额">
            <el-input v-model="form.balance" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="币种名">
            <el-input v-model="form.coin" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="账户类型">
            <el-select v-model="form.accountType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.account_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="钱包地址">
            <el-input v-model="form.walletAddress" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建人">
            <el-input v-model="form.createBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="更新人">
            <el-input v-model="form.updateBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-input v-model="form.createTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="更新时间">
            <el-input v-model="form.updateTime" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="id" />
        <el-table-column prop="userId" label="用户ID" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="userType" label="用户类型">
          <template slot-scope="scope">
            {{ dict.label.user_type[scope.row.userType] }}
          </template>
        </el-table-column>
        <el-table-column prop="remark" label="用户备注" />
        <el-table-column prop="balance" label="钱包余额" />
        <el-table-column prop="coin" label="币种名" />
        <el-table-column prop="accountType" label="账户类型">
          <template slot-scope="scope">
            {{ dict.label.account_type[scope.row.accountType] }}
          </template>
        </el-table-column>
        <el-table-column prop="walletAddress" label="钱包地址" />
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="updateBy" label="更新人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column v-if="checkPer(['admin','userWallet:edit','userWallet:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudUserWallet from '@/api/userWallet'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, username: null, userType: null, remark: null, balance: null, usedMarginBalance: null, bonusBalance: null, loanAmount: null, frozenBalance: null, coin: null, accountType: null, walletAddress: null, createBy: null, updateBy: null, createTime: null, updateTime: null, version: null }
export default {
  name: 'UserWallet',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['user_type', 'account_type'],
  cruds() {
    return CRUD({ title: '钱包管理', url: 'api/userWallet', idField: 'id', sort: 'id,desc', crudMethod: { ...crudUserWallet }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'userWallet:add'],
        edit: ['admin', 'userWallet:edit'],
        del: ['admin', 'userWallet:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'userId', display_name: '用户ID' },
        { key: 'username', display_name: '用户名' },
        { key: 'userType', display_name: '用户类型' },
        { key: 'coin', display_name: '币种名' },
        { key: 'accountType', display_name: '账户类型' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
