<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">交易对符号</label>
        <el-input v-model="query.symbol" clearable placeholder="交易对符号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
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
          <el-form-item label="产品名称" prop="productName">
            <el-input v-model="form.productName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交易对符号" prop="symbol">
            <el-input v-model="form.symbol" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="市场代码" prop="marketCode">
            <el-input v-model="form.marketCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="原价格">
            <el-input v-model="form.originalPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="当前价格" prop="currentPrice">
            <el-input v-model="form.currentPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="涨跌百分比">
            <el-input v-model="form.changeRate" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="涨跌状态(1=涨,0=平,-1=跌)">
            <el-select v-model="form.changeStatus" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.change_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="场控类型(1=自动,2=盈亏,3=手动)" prop="controlType">
            <el-select v-model="form.controlType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.banner_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="启动延时(秒)" prop="startDelaySeconds">
            <el-input v-model="form.startDelaySeconds" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="预设时长(秒)" prop="presetDurationSeconds">
            <el-input v-model="form.presetDurationSeconds" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="回归时长(秒)" prop="revertDurationSeconds">
            <el-input v-model="form.revertDurationSeconds" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="停留时长(秒)" prop="stayDurationSeconds">
            <el-input v-model="form.stayDurationSeconds" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="修改价格" prop="modifyPrice">
            <el-input v-model="form.modifyPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="最后修改人">
            <el-input v-model="form.updateBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="状态(0=禁用, 1=启用)" prop="status">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="创建人">
            <el-input v-model="form.createBy" style="width: 370px;" />
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
        <el-table-column prop="productName" label="产品名称" />
        <el-table-column prop="symbol" label="交易对符号" />
        <el-table-column prop="marketCode" label="市场代码" />
        <el-table-column prop="originalPrice" label="原价格" />
        <el-table-column prop="currentPrice" label="当前价格" />
        <el-table-column prop="changeRate" label="涨跌百分比" />
        <el-table-column prop="changeStatus" label="涨跌状态(1=涨,0=平,-1=跌)">
          <template slot-scope="scope">
            {{ dict.label.change_status[scope.row.changeStatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="controlType" label="场控类型(1=自动,2=盈亏,3=手动)">
          <template slot-scope="scope">
            {{ dict.label.banner_type[scope.row.controlType] }}
          </template>
        </el-table-column>
        <el-table-column prop="startDelaySeconds" label="启动延时(秒)" />
        <el-table-column prop="presetDurationSeconds" label="预设时长(秒)" />
        <el-table-column prop="revertDurationSeconds" label="回归时长(秒)" />
        <el-table-column prop="stayDurationSeconds" label="停留时长(秒)" />
        <el-table-column prop="modifyPrice" label="修改价格" />
        <el-table-column prop="updateBy" label="最后修改人" />
        <el-table-column prop="status" label="状态(0=禁用, 1=启用)">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column v-if="checkPer(['admin','marketContractCfg:edit','marketContractCfg:del'])" label="操作" width="150px" align="center">
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
import crudMarketContractCfg from '@/api/marketContractCfg'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, productName: null, symbol: null, marketCode: null, originalPrice: null, currentPrice: null, changeRate: null, changeStatus: null, controlType: null, startDelaySeconds: null, presetDurationSeconds: null, revertDurationSeconds: null, stayDurationSeconds: null, modifyPrice: null, updateBy: null, status: null, createBy: null, createTime: null, updateTime: null }
export default {
  name: 'MarketContractCfg',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['change_status', 'banner_type', 'status'],
  cruds() {
    return CRUD({ title: '加密场控', url: 'api/marketContractCfg', idField: 'id', sort: 'id,desc', crudMethod: { ...crudMarketContractCfg }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'marketContractCfg:add'],
        edit: ['admin', 'marketContractCfg:edit'],
        del: ['admin', 'marketContractCfg:del']
      },
      rules: {
        productName: [
          { required: true, message: '产品名称不能为空', trigger: 'blur' }
        ],
        symbol: [
          { required: true, message: '交易对符号不能为空', trigger: 'blur' }
        ],
        marketCode: [
          { required: true, message: '市场代码不能为空', trigger: 'blur' }
        ],
        currentPrice: [
          { required: true, message: '当前价格不能为空', trigger: 'blur' }
        ],
        controlType: [
          { required: true, message: '场控类型(1=自动,2=盈亏,3=手动)不能为空', trigger: 'blur' }
        ],
        startDelaySeconds: [
          { required: true, message: '启动延时(秒)不能为空', trigger: 'blur' }
        ],
        presetDurationSeconds: [
          { required: true, message: '预设时长(秒)不能为空', trigger: 'blur' }
        ],
        revertDurationSeconds: [
          { required: true, message: '回归时长(秒)不能为空', trigger: 'blur' }
        ],
        stayDurationSeconds: [
          { required: true, message: '停留时长(秒)不能为空', trigger: 'blur' }
        ],
        modifyPrice: [
          { required: true, message: '修改价格不能为空', trigger: 'blur' }
        ],
        status: [
          { required: true, message: '状态(0=禁用, 1=启用)不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'symbol', display_name: '交易对符号' }
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
