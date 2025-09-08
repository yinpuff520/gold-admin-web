<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">登录用户ID</label>
        <el-input v-model="query.userId" clearable placeholder="登录用户ID" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">登录用户名</label>
        <el-input v-model="query.username" clearable placeholder="登录用户名" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">登录状态（0成功 1失败）</label>
        <el-input v-model="query.status" clearable placeholder="登录状态（0成功 1失败）" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.loginTime"
          start-placeholder="loginTimeStart"
          end-placeholder="loginTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="登录用户ID">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="登录用户名">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="访问IP">
            <el-input v-model="form.ipaddr" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="访问位置">
            <el-input v-model="form.loginLocation" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="浏览器">
            <el-input v-model="form.browser" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="系统OS">
            <el-input v-model="form.os" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="登录状态（0成功 1失败）">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.login_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="提示信息">
            <el-input v-model="form.msg" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="访问时间">
            <el-input v-model="form.loginTime" style="width: 370px;" />
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
        <el-table-column prop="userId" label="登录用户ID" />
        <el-table-column prop="username" label="登录用户名" />
        <el-table-column prop="ipaddr" label="访问IP" />
        <el-table-column prop="loginLocation" label="访问位置" />
        <el-table-column prop="browser" label="浏览器" />
        <el-table-column prop="os" label="系统OS" />
        <el-table-column prop="status" label="登录状态">
          <template slot-scope="scope">
            {{ dict.label.login_status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="msg" label="提示信息" />
        <el-table-column prop="loginTime" label="访问时间" />
<!--        <el-table-column v-if="checkPer(['admin','memberLoginLog:edit','memberLoginLog:del'])" label="操作" width="150px" align="center">-->
<!--          <template slot-scope="scope">-->
<!--            <udOperation-->
<!--              :data="scope.row"-->
<!--              :permission="permission"-->
<!--            />-->
<!--          </template>-->
<!--        </el-table-column>-->
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudMemberLoginLog from '@/api/memberLoginLog'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, username: null, ipaddr: null, loginLocation: null, browser: null, os: null, status: null, msg: null, loginTime: null }
export default {
  name: 'MemberLoginLog',
  // eslint-disable-next-line vue/no-unused-components
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['login_status'],
  cruds() {
    return CRUD({ title: '会员登录日志管理', url: 'api/memberLoginLog', idField: 'id', sort: 'id,desc', crudMethod: { ...crudMemberLoginLog }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'memberLoginLog:add'],
        edit: ['admin', 'memberLoginLog:edit'],
        del: ['admin', 'memberLoginLog:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'userId', display_name: '登录用户ID' },
        { key: 'username', display_name: '登录用户名' },
        { key: 'status', display_name: '登录状态（0成功 1失败）' }
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
