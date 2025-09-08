<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">用户ID</label>
        <el-input v-model="query.userId" clearable placeholder="用户ID" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">认证状态</label>
        <el-input v-model="query.status" clearable placeholder="认证状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="主键ID">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户ID">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="真实姓名">
            <el-input v-model="form.realName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="证件类型">
            <el-select v-model="form.idType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.id_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="证件号码">
            <el-input v-model="form.idNumber" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="国家代码">
            <el-input v-model="form.countryCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="性别">
            <el-select v-model="form.gender" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.gender"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="生日">
            <el-input v-model="form.birthday" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="证件正面照片">
            <el-input v-model="form.idFrontUrl" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="证件反面照片">
            <el-input v-model="form.idBackUrl" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="手持证件照片">
            <el-input v-model="form.idHandheldUrl" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="认证状态">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.real_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="审核备注">
            <el-input v-model="form.remark" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建人用户ID">
            <el-input v-model="form.createBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="更新人用户ID">
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
        <el-table-column prop="id" label="主键ID" />
        <el-table-column prop="userId" label="用户ID" />
        <el-table-column prop="realName" label="真实姓名" />
        <el-table-column prop="idType" label="证件类型">
          <template slot-scope="scope">
            {{ dict.label.id_type[scope.row.idType] }}
          </template>
        </el-table-column>
        <el-table-column prop="idNumber" label="证件号码" />
        <el-table-column prop="countryCode" label="国家代码" />
        <el-table-column prop="gender" label="性别">
          <template slot-scope="scope">
            {{ dict.label.gender[scope.row.gender] }}
          </template>
        </el-table-column>
        <el-table-column prop="birthday" label="生日" />
        <el-table-column prop="idFrontUrl" label="证件正面照片" />
        <el-table-column prop="idBackUrl" label="证件反面照片" />
        <el-table-column prop="idHandheldUrl" label="手持证件照片" />
        <el-table-column prop="status" label="认证状态">
          <template slot-scope="scope">
            {{ dict.label.real_status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="remark" label="审核备注" />
        <el-table-column prop="createBy" label="创建人用户ID" />
        <el-table-column prop="updateBy" label="更新人用户ID" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column v-if="checkPer(['admin','userRealName:edit','userRealName:del'])" label="操作" width="150px" align="center">
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
import crudUserRealName from '@/api/userRealName'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, realName: null, idType: null, idNumber: null, countryCode: null, gender: null, birthday: null, idFrontUrl: null, idBackUrl: null, idHandheldUrl: null, status: null, remark: null, createBy: null, updateBy: null, createTime: null, updateTime: null }
export default {
  name: 'UserRealName',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['id_type', 'gender', 'real_status'],
  cruds() {
    return CRUD({ title: '会员审核', url: 'api/userRealName', idField: 'id', sort: 'id,desc', crudMethod: { ...crudUserRealName }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'userRealName:add'],
        edit: ['admin', 'userRealName:edit'],
        del: ['admin', 'userRealName:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'userId', display_name: '用户ID，关联 user 表' },
        { key: 'status', display_name: '认证状态' }
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
