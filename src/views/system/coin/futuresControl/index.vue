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
          <el-form-item label="用户ID" prop="userId">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户名" prop="username">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户类型" prop="userType">
            <el-select v-model="form.userType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.user_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="期权时间" prop="optionTime">
            <el-input v-model="form.optionTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="盈利百分比">
            <el-input v-model="form.profitRate" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="亏损百分比">
            <el-input v-model="form.lossRate" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="市场/合约代码" prop="marketCode">
            <el-input v-model="form.marketCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交易符号" prop="symbol">
            <el-input v-model="form.symbol" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交易金额">
            <el-input v-model="form.tradeAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="最小交易量">
            <el-input v-model="form.minAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="最大交易量">
            <el-input v-model="form.maxAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="价格">
            <el-input v-model="form.price" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交易方向" prop="sideType">
            <el-select v-model="form.sideType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.side_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="输赢状态" prop="winStatus">
            <el-select v-model="form.winStatus" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.win_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="购买类型" prop="purchaseType">
            <el-select v-model="form.purchaseType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.purchase_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="盈利金额">
            <el-input v-model="form.profit" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否加仓订单" prop="isAdditionalOrder">
            <el-select v-model="form.isAdditionalOrder" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.is_additional_order"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="场控类型" prop="controlType">
            <el-select v-model="form.controlType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.change_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="创建人">
            <el-input v-model="form.createBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="修改人">
            <el-input v-model="form.updateBy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-input v-model="form.createTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="修改时间">
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
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="userType" label="用户类型">
          <template slot-scope="scope">
            {{ dict.label.user_type[scope.row.userType] }}
          </template>
        </el-table-column>
        <el-table-column prop="optionTime" label="期权时间" />
        <el-table-column prop="profitRate" label="盈利百分比" />
        <el-table-column prop="lossRate" label="亏损百分比" />
        <el-table-column prop="marketCode" label="市场/合约代码" />
        <el-table-column prop="symbol" label="交易符号" />
        <el-table-column prop="tradeAmount" label="交易金额" />
        <el-table-column prop="minAmount" label="最小交易量" />
        <el-table-column prop="maxAmount" label="最大交易量" />
        <el-table-column prop="price" label="价格" />
        <el-table-column prop="sideType" label="交易方向">
          <template slot-scope="scope">
            {{ dict.label.side_type[scope.row.sideType] }}
          </template>
        </el-table-column>
        <el-table-column prop="winStatus" label="输赢状态">
          <template slot-scope="scope">
            {{ dict.label.win_status[scope.row.winStatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="purchaseType" label="购买类型">
          <template slot-scope="scope">
            {{ dict.label.purchase_type[scope.row.purchaseType] }}
          </template>
        </el-table-column>
        <el-table-column prop="profit" label="盈利金额" />
        <el-table-column prop="isAdditionalOrder" label="是否加仓订单">
          <template slot-scope="scope">
            {{ dict.label.is_additional_order[scope.row.isAdditionalOrder] }}
          </template>
        </el-table-column>
        <el-table-column prop="controlType" label="场控类型">
          <template slot-scope="scope">
            {{ dict.label.change_type[scope.row.controlType] }}
          </template>
        </el-table-column>
        <el-table-column prop="createBy" label="创建人" />
        <el-table-column prop="updateBy" label="修改人" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="修改时间" />
        <el-table-column v-if="checkPer(['admin','marketOptionCfg:edit','marketOptionCfg:del'])" label="操作" width="150px" align="center">
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
import crudMarketOptionCfg from '@/api/marketOptionCfg'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, username: null, userType: null, optionTime: null, profitRate: null, lossRate: null, marketCode: null, symbol: null, tradeAmount: null, minAmount: null, maxAmount: null, price: null, sideType: null, winStatus: null, purchaseType: null, profit: null, isAdditionalOrder: null, controlType: null, createBy: null, updateBy: null, createTime: null, updateTime: null }
export default {
  name: 'MarketOptionCfg',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['user_type', 'side_type', 'win_status', 'purchase_type', 'is_additional_order', 'change_type'],
  cruds() {
    return CRUD({ title: '期货场控', url: 'api/marketOptionCfg', idField: 'id', sort: 'id,desc', crudMethod: { ...crudMarketOptionCfg }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'marketOptionCfg:add'],
        edit: ['admin', 'marketOptionCfg:edit'],
        del: ['admin', 'marketOptionCfg:del']
      },
      rules: {
        userId: [
          { required: true, message: '用户ID不能为空', trigger: 'blur' }
        ],
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' }
        ],
        userType: [
          { required: true, message: '用户类型不能为空', trigger: 'blur' }
        ],
        optionTime: [
          { required: true, message: '期权时间不能为空', trigger: 'blur' }
        ],
        marketCode: [
          { required: true, message: '市场/合约代码不能为空', trigger: 'blur' }
        ],
        symbol: [
          { required: true, message: '交易符号不能为空', trigger: 'blur' }
        ],
        sideType: [
          { required: true, message: '交易方向不能为空', trigger: 'blur' }
        ],
        winStatus: [
          { required: true, message: '输赢状态不能为空', trigger: 'blur' }
        ],
        purchaseType: [
          { required: true, message: '购买类型不能为空', trigger: 'blur' }
        ],
        isAdditionalOrder: [
          { required: true, message: '是否加仓订单不能为空', trigger: 'blur' }
        ],
        controlType: [
          { required: true, message: '场控类型不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'userId', display_name: '用户ID' },
        { key: 'username', display_name: '用户名' },
        { key: 'userType', display_name: '用户类型' }
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
