<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">id</label>
        <el-input v-model="query.id" clearable placeholder="id" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">用户ID</label>
        <el-input v-model="query.userId" clearable placeholder="用户ID" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">提现方式</label>
        <el-input v-model="query.withdrawStatus" clearable placeholder="提现方式" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">审核状态</label>
        <el-input v-model="query.auditStatus" clearable placeholder="审核状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.withdrawTime"
          start-placeholder="withdrawTimeStart"
          end-placeholder="withdrawTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="id" prop="id">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户ID" prop="userId">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="提现币种" prop="coin">
            <el-input v-model="form.coin" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="提现金额" prop="amount">
            <el-input v-model="form.amount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="手续费">
            <el-input v-model="form.fee" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="提现时间" prop="withdrawTime">
            <el-input v-model="form.withdrawTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="提现方式" prop="withdrawStatus">
            <el-select v-model="form.withdrawStatus" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.withdraw_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="审核状态" prop="auditStatus">
            <el-select v-model="form.auditStatus" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.audit_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
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
        <el-table-column prop="coin" label="提现币种" />
        <el-table-column prop="amount" label="提现金额" />
        <el-table-column prop="fee" label="手续费" />
        <el-table-column prop="withdrawTime" label="提现时间" />
        <el-table-column prop="withdrawStatus" label="提现方式">
          <template slot-scope="scope">
            {{ dict.label.withdraw_status[scope.row.withdrawStatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="auditStatus" label="审核状态">
          <template slot-scope="scope">
            {{ dict.label.audit_status[scope.row.auditStatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="updateBy" label="更新人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column v-if="checkPer(['admin','payWithdrawOrder:edit','payWithdrawOrder:del'])" label="操作" width="150px" align="center">
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
import crudPayWithdrawOrder from '@/api/payWithdrawOrder'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, coin: null, amount: null, fee: null, withdrawTime: null, withdrawStatus: null, auditStatus: null, createBy: null, updateBy: null, createTime: null, updateTime: null }
export default {
  name: 'PayWithdrawOrder',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['withdraw_status', 'audit_status'],
  cruds() {
    return CRUD({ title: '提现管理', url: 'api/payWithdrawOrder', idField: 'id', sort: 'id,desc', crudMethod: { ...crudPayWithdrawOrder }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'payWithdrawOrder:add'],
        edit: ['admin', 'payWithdrawOrder:edit'],
        del: ['admin', 'payWithdrawOrder:del']
      },
      rules: {
        id: [
          { required: true, message: 'id不能为空', trigger: 'blur' }
        ],
        userId: [
          { required: true, message: '用户ID不能为空', trigger: 'blur' }
        ],
        coin: [
          { required: true, message: '提现币种不能为空', trigger: 'blur' }
        ],
        amount: [
          { required: true, message: '提现金额不能为空', trigger: 'blur' }
        ],
        withdrawTime: [
          { required: true, message: '提现时间不能为空', trigger: 'blur' }
        ],
        withdrawStatus: [
          { required: true, message: '提现方式不能为空', trigger: 'blur' }
        ],
        auditStatus: [
          { required: true, message: '审核状态不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'id', display_name: 'id' },
        { key: 'userId', display_name: '用户ID' },
        { key: 'withdrawStatus', display_name: '提现方式' },
        { key: 'auditStatus', display_name: '审核状态' }
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
