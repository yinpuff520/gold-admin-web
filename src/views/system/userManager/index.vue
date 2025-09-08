<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">用户UID</label>
        <el-input v-model="query.uid" clearable placeholder="用户UID" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">用户类型</label>
        <el-input v-model="query.userType" clearable placeholder="用户类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">手机号</label>
        <el-input v-model="query.phone" clearable placeholder="手机号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">邮箱</label>
        <el-input v-model="query.email" clearable placeholder="邮箱" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">邀请码</label>
        <el-input v-model="query.inviteCode" clearable placeholder="邀请码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">申请码</label>
        <el-input v-model="query.applyCode" clearable placeholder="申请码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.createTime"
          start-placeholder="createTimeStart"
          end-placeholder="createTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="用户UID" prop="uid">
            <el-input v-model="form.uid" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户名">
            <el-input v-model="form.userName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="登录名">
            <el-input v-model="form.loginName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交易密码">
            <el-input v-model="form.tradePassword" style="width: 370px;" />
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
          <el-form-item label="用户状态">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="VIP等级">
            <el-input v-model="form.vipLevel" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否可交易">
            <el-select v-model="form.tradable" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="是否企业账户">
            <el-select v-model="form.isEnterprise" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="手机号">
            <el-input v-model="form.phone" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="邮箱">
            <el-input v-model="form.email" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="国家区号">
            <el-input v-model="form.countryCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="上级用户ID">
            <el-input v-model="form.parentUserId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="上级代理用户名">
            <el-input v-model="form.parentAgentUsername" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="代理用户名">
            <el-input v-model="form.agentUsername" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="代理类型">
            <el-input v-model="form.agentType" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="归属员">
            <el-input v-model="form.owner" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="注册时间">
            <el-input v-model="form.registerTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="注册IP">
            <el-input v-model="form.registerIp" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="注册域名">
            <el-input v-model="form.host" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="最后登录时间">
            <el-input v-model="form.lastLoginTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="最后登录IP">
            <el-input v-model="form.lastLoginIp" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="登录失败次数">
            <el-input v-model="form.loginFailCount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="账户锁定到期时间">
            <el-input v-model="form.lockedUntil" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="钱包地址">
            <el-input v-model="form.walletAddress" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="地址类型">
            <el-input v-model="form.walletType" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="钱包登录地址">
            <el-input v-model="form.address" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="借款额度">
            <el-input v-model="form.loanLimit" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="借款次数">
            <el-input v-model="form.loanCount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="提款剩余所需打码量">
            <el-input v-model="form.withdrawCodeRequired" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="邀请码">
            <el-input v-model="form.inviteCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="申请码">
            <el-input v-model="form.applyCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户头像">
            <el-input v-model="form.avatar" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户来源">
            <el-input v-model="form.source" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户备注">
            <el-input v-model="form.remark" style="width: 370px;" />
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
        <el-table-column prop="uid" label="用户UID" />
        <el-table-column prop="userName" label="用户名" />
        <el-table-column prop="loginName" label="登录名" />
        <el-table-column prop="tradePassword" label="交易密码" />
        <el-table-column prop="userType" label="用户类型">
          <template slot-scope="scope">
            {{ dict.label.user_type[scope.row.userType] }}
          </template>
        </el-table-column>
        <el-table-column prop="status" label="用户状态">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="vipLevel" label="VIP等级" />
        <el-table-column prop="tradable" label="是否可交易">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.tradable] }}
          </template>
        </el-table-column>
        <el-table-column prop="isEnterprise" label="是否企业账户">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isEnterprise] }}
          </template>
        </el-table-column>
        <el-table-column prop="phone" label="手机号" />
        <el-table-column prop="email" label="邮箱" />
        <el-table-column prop="countryCode" label="国家区号" />
        <el-table-column prop="parentUserId" label="上级用户ID" />
        <el-table-column prop="parentAgentUsername" label="上级代理用户名" />
        <el-table-column prop="agentUsername" label="代理用户名" />
        <el-table-column prop="agentType" label="代理类型" />
        <el-table-column prop="owner" label="归属员" />
        <el-table-column prop="registerTime" label="注册时间" />
        <el-table-column prop="registerIp" label="注册IP" />
        <el-table-column prop="host" label="注册域名" />
        <el-table-column prop="lastLoginTime" label="最后登录时间" />
        <el-table-column prop="lastLoginIp" label="最后登录IP" />
        <el-table-column prop="loginFailCount" label="登录失败次数" />
        <el-table-column prop="lockedUntil" label="账户锁定到期时间" />
        <el-table-column prop="walletAddress" label="钱包地址" />
        <el-table-column prop="walletType" label="地址类型" />
        <el-table-column prop="address" label="钱包登录地址" />
        <el-table-column prop="loanLimit" label="借款额度" />
        <el-table-column prop="loanCount" label="借款次数" />
        <el-table-column prop="withdrawCodeRequired" label="提款剩余所需打码量" />
        <el-table-column prop="inviteCode" label="邀请码" />
        <el-table-column prop="applyCode" label="申请码" />
        <el-table-column prop="avatar" label="用户头像" />
        <el-table-column prop="source" label="用户来源" />
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="updateBy" label="更新人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column prop="remark" label="用户备注" />
        <el-table-column v-if="checkPer(['admin','user:edit','user:del'])" label="操作" width="150px" align="center">
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
import crudUser from '@/api/user'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, uid: null, userName: null, loginName: null, password: null, tradePassword: null, salt: null, userType: null, status: null, vipLevel: null, tradable: null, isEnterprise: null, phone: null, email: null, countryCode: null, parentUserId: null, parentAgentUsername: null, agentUsername: null, agentType: null, owner: null, registerTime: null, registerIp: null, host: null, lastLoginTime: null, lastLoginIp: null, loginFailCount: null, lockedUntil: null, walletAddress: null, walletType: null, address: null, loanLimit: null, loanCount: null, withdrawCodeRequired: null, rechargeKey: null, inviteCode: null, applyCode: null, avatar: null, source: null, createBy: null, updateBy: null, createTime: null, updateTime: null, remark: null, id: null, uid: null, userName: null, loginName: null, password: null, tradePassword: null, salt: null, userType: null, status: null, vipLevel: null, tradable: null, isEnterprise: null, phone: null, email: null, countryCode: null, parentUserId: null, parentAgentUsername: null, agentUsername: null, agentType: null, owner: null, registerTime: null, registerIp: null, host: null, lastLoginTime: null, lastLoginIp: null, loginFailCount: null, lockedUntil: null, walletAddress: null, walletType: null, address: null, loanLimit: null, loanCount: null, withdrawCodeRequired: null, rechargeKey: null, inviteCode: null, applyCode: null, avatar: null, source: null, createBy: null, updateBy: null, createTime: null, updateTime: null, remark: null, blackStatus: null }
export default {
  name: 'User',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['user_type', 'status', 'user_status'],
  cruds() {
    return CRUD({ title: '会员管理', url: 'api/user', idField: 'id', sort: 'id,desc', crudMethod: { ...crudUser }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'user:add'],
        edit: ['admin', 'user:edit'],
        del: ['admin', 'user:del']
      },
      rules: {
        uid: [
          { required: true, message: '用户UID不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'uid', display_name: '用户UID' },
        { key: 'userType', display_name: '用户类型' },
        { key: 'phone', display_name: '手机号' },
        { key: 'email', display_name: '邮箱' },
        { key: 'inviteCode', display_name: '邀请码' },
        { key: 'applyCode', display_name: '申请码' }
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
