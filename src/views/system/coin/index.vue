<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">交易对符号 (例如 BTC/USDT)</label>
        <el-input v-model="query.symbol" clearable placeholder="交易对符号 (例如 BTC/USDT)" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
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
          <el-form-item label="交易对符号 (例如 BTC/USDT)" prop="symbol">
            <el-input v-model="form.symbol" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基础币种" prop="baseCoinCode">
            <el-input v-model="form.baseCoinCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="计价币种" prop="quoteCoinCode">
            <el-input v-model="form.quoteCoinCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="金额小数位数" prop="numScale">
            <el-input v-model="form.numScale" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="价格小数位数" prop="priceScale">
            <el-input v-model="form.priceScale" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="自定义图标">
            <el-input v-model="form.icon" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否发布" prop="isPublished">
            <el-select v-model="form.isPublished" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="是否锁仓现货买入" prop="isLockSpotBuy">
            <el-select v-model="form.isLockSpotBuy" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="是否锁仓现货卖出" prop="isLockSpotSell">
            <el-select v-model="form.isLockSpotSell" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="是否锁仓合约" prop="isLockContract">
            <el-select v-model="form.isLockContract" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="现货买入锁仓开始时间">
            <el-input v-model="form.spotBuyLockStart" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="现货买入锁仓时间(天)">
            <el-input v-model="form.spotBuyLockDays" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="现货卖出锁仓时间(天)">
            <el-input v-model="form.spotSellLockDays" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="合约价值">
            <el-input v-model="form.contractValue" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否热门" prop="isHot">
            <el-select v-model="form.isHot" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="排序">
            <el-input v-model="form.sort" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="别名">
            <el-input v-model="form.alias" style="width: 370px;" />
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
        <el-table-column prop="symbol" label="交易对符号 (例如 BTC/USDT)" />
        <el-table-column prop="baseCoinCode" label="基础币种" />
        <el-table-column prop="quoteCoinCode" label="计价币种" />
        <el-table-column prop="numScale" label="金额小数位数" />
        <el-table-column prop="priceScale" label="价格小数位数" />
        <el-table-column prop="icon" label="自定义图标" />
        <el-table-column prop="isPublished" label="是否发布">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isPublished] }}
          </template>
        </el-table-column>
        <el-table-column prop="isLockSpotBuy" label="是否锁仓现货买入">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isLockSpotBuy] }}
          </template>
        </el-table-column>
        <el-table-column prop="isLockSpotSell" label="是否锁仓现货卖出">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isLockSpotSell] }}
          </template>
        </el-table-column>
        <el-table-column prop="isLockContract" label="是否锁仓合约">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isLockContract] }}
          </template>
        </el-table-column>
        <el-table-column prop="spotBuyLockStart" label="现货买入锁仓开始时间" />
        <el-table-column prop="spotBuyLockDays" label="现货买入锁仓时间(天)" />
        <el-table-column prop="spotSellLockDays" label="现货卖出锁仓时间(天)" />
        <el-table-column prop="contractValue" label="合约价值" />
        <el-table-column prop="isHot" label="是否热门">
          <template slot-scope="scope">
            {{ dict.label.status[scope.row.isHot] }}
          </template>
        </el-table-column>
        <el-table-column prop="sort" label="排序" />
        <el-table-column prop="alias" label="别名" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="修改时间" />
        <el-table-column v-if="checkPer(['admin','marketCoin:edit','marketCoin:del'])" label="操作" width="150px" align="center">
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
import crudMarketCoin from '@/api/marketCoin'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, symbol: null, baseCoinCode: null, quoteCoinCode: null, numScale: null, priceScale: null, icon: null, isPublished: null, isLockSpotBuy: null, isLockSpotSell: null, isLockContract: null, spotBuyLockStart: null, spotBuyLockDays: null, spotSellLockDays: null, contractValue: null, isHot: null, sort: null, alias: null, createBy: null, createTime: null, updateBy: null, updateTime: null }
export default {
  name: 'MarketCoin',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['status'],
  cruds() {
    return CRUD({ title: '加密货币币种列表', url: 'api/marketCoin', idField: 'id', sort: 'id,desc', crudMethod: { ...crudMarketCoin }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'marketCoin:add'],
        edit: ['admin', 'marketCoin:edit'],
        del: ['admin', 'marketCoin:del']
      },
      rules: {
        symbol: [
          { required: true, message: '交易对符号 (例如 BTC/USDT)不能为空', trigger: 'blur' }
        ],
        baseCoinCode: [
          { required: true, message: '基础币种不能为空', trigger: 'blur' }
        ],
        quoteCoinCode: [
          { required: true, message: '计价币种不能为空', trigger: 'blur' }
        ],
        numScale: [
          { required: true, message: '金额小数位数不能为空', trigger: 'blur' }
        ],
        priceScale: [
          { required: true, message: '价格小数位数不能为空', trigger: 'blur' }
        ],
        isPublished: [
          { required: true, message: '是否发布不能为空', trigger: 'blur' }
        ],
        isLockSpotBuy: [
          { required: true, message: '是否锁仓现货买入不能为空', trigger: 'blur' }
        ],
        isLockSpotSell: [
          { required: true, message: '是否锁仓现货卖出不能为空', trigger: 'blur' }
        ],
        isLockContract: [
          { required: true, message: '是否锁仓合约不能为空', trigger: 'blur' }
        ],
        isHot: [
          { required: true, message: '是否热门不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'symbol', display_name: '交易对符号 (例如 BTC/USDT)' }
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
