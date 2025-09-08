<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="标题" prop="bannerTitle">
            <el-input v-model="form.bannerTitle" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="内容链接" prop="bannerUrl">
            <el-input v-model="form.bannerUrl" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="使用类型" prop="bannerType">
            <el-select v-model="form.bannerType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.banner_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="跳转地址" prop="jumpUrl">
            <el-input v-model="form.jumpUrl" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="排序(越大的排在越前面)" prop="isOrder">
            <el-input v-model="form.isOrder" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否启用" prop="bannerStatus">
            <el-select v-model="form.bannerStatus" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.banner_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
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
        <el-table-column prop="id" label="ID" />
        <el-table-column prop="bannerTitle" label="标题" />
        <el-table-column prop="bannerUrl" label="内容链接" />
        <el-table-column prop="bannerType" label="使用类型">
          <template slot-scope="scope">
            {{ dict.label.banner_type[scope.row.bannerType] }}
          </template>
        </el-table-column>
        <el-table-column prop="jumpUrl" label="跳转地址" />
        <el-table-column prop="isOrder" label="排序(越大的排在越前面)" />
        <el-table-column prop="bannerStatus" label="是否启用">
          <template slot-scope="scope">
            {{ dict.label.banner_status[scope.row.bannerStatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="updateBy" label="更新人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column v-if="checkPer(['admin','sysBanner:edit','sysBanner:del'])" label="操作" width="150px" align="center">
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
import crudSysBanner from '@/api/sysBanner'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, bannerTitle: null, bannerUrl: null, bannerType: null, jumpUrl: null, isOrder: null, bannerStatus: null, createBy: null, updateBy: null, createTime: null, updateTime: null }
export default {
  name: 'SysBanner',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['banner_type', 'banner_status'],
  cruds() {
    return CRUD({ title: 'banner管理', url: 'api/sysBanner', idField: 'id', sort: 'id,desc', crudMethod: { ...crudSysBanner }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'sysBanner:add'],
        edit: ['admin', 'sysBanner:edit'],
        del: ['admin', 'sysBanner:del']
      },
      rules: {
        id: [
          { required: true, message: 'ID不能为空', trigger: 'blur' }
        ],
        bannerTitle: [
          { required: true, message: '标题不能为空', trigger: 'blur' }
        ],
        bannerUrl: [
          { required: true, message: '内容链接不能为空', trigger: 'blur' }
        ],
        bannerType: [
          { required: true, message: '使用类型不能为空', trigger: 'blur' }
        ],
        jumpUrl: [
          { required: true, message: '跳转地址不能为空', trigger: 'blur' }
        ],
        isOrder: [
          { required: true, message: '排序(越大的排在越前面)不能为空', trigger: 'blur' }
        ],
        bannerStatus: [
          { required: true, message: '是否启用不能为空', trigger: 'blur' }
        ]
      }    }
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
