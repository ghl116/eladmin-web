<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog
        :close-on-click-modal="false"
        :before-close="crud.cancelCU"
        :visible.sync="crud.status.cu > 0"
        :title="crud.status.title"
        width="500px"
      >
        <el-form
          ref="form"
          :model="form"
          :rules="rules"
          size="small"
          label-width="80px"
        >

          <el-form-item label="租户编号" prop="zhbh">
            <el-select
              v-model="form.zhbh"
              style="width: 370px"
              placeholder="请选择"
            >
              <el-option
                v-for="item in allCust"
                :key="item.zhbh"
                :label="item.khmc"
                :value="item.zhbh"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="产品id" prop="productid">
            <el-select
              v-model="form.productid"
              style="width: 370px"
              placeholder="请选择"
            >
              <el-option
                v-for="item in allProduct"
                :key="item.id"
                :label="item.productname"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="商务阶段" prop="businessstatus">
            <el-select
              v-model="form.businessstatus"
              filterable
              placeholder="请选择"
            >
              <el-option
                v-for="item in dict.businessStatus"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="上线情况">
            <el-input v-model="form.online" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="上线日期">
            <el-date-picker
              v-model="form.onlinedate"
              type="datetime"
              style="width: 370px;"
            />
          </el-form-item>
          <el-form-item label="下线日期">
            <el-date-picker
              v-model="form.offlinedate"
              type="datetime"
              style="width: 370px;"
            />
          </el-form-item>
          <el-form-item label="上线负责人">
            <el-input v-model="form.onlineleader" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="商务岗确认人">
            <el-input v-model="form.businessman" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="需求工单号">
            <el-input v-model="form.reqnum" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="合同编号">
            <el-input v-model="form.contractnum" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="建设费">
            <el-input v-model="form.constructioncost" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="运维费/年">
            <el-input v-model="form.maintenancecost" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="备注">
            <el-input
              v-model="form.note"
              :rows="3"
              type="textarea"
              style="width: 370px;"
            />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button
            :loading="crud.cu === 2"
            type="primary"
            @click="crud.submitCU"
          >确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table
        ref="table"
        v-loading="crud.loading"
        :data="crud.data"
        size="small"
        style="width: 100%;"
        @selection-change="crud.selectionChangeHandler"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column prop="zhbh" label="租户编号">
          <template slot-scope="scope">
            {{ getCustName(scope.row.zhbh) }}
          </template>
        </el-table-column>
        <el-table-column prop="productid" label="产品id">
          <template slot-scope="scope">
            {{ getProductName(scope.row.productid) }}
          </template>
        </el-table-column>
        <el-table-column prop="businessstatus" label="商务阶段">
          <template slot-scope="scope">
            {{ dict.label.businessStatus[scope.row.businessstatus] }}
          </template>
        </el-table-column>
        <el-table-column prop="online" label="上线情况" />
        <el-table-column prop="onlinedate" label="上线日期">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.onlinedate) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="offlinedate" label="下线日期">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.offlinedate) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="onlineleader" label="上线负责人" />
        <el-table-column prop="businessman" label="商务岗确认人" />
        <el-table-column prop="reqnum" label="需求工单号" />
        <el-table-column prop="contractnum" label="合同编号" />
        <el-table-column prop="constructioncost" label="建设费" />
        <el-table-column prop="maintenancecost" label="运维费/年" />
        <el-table-column prop="note" label="备注" />
        <el-table-column prop="createBy" label="创建者" />
        <el-table-column prop="updatedBy" label="更新者" />
        <el-table-column prop="createTime" label="创建日期">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="更新时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.updateTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column
          v-permission="['admin', 'salesdetail:edit', 'salesdetail:del']"
          label="操作"
          width="150px"
          align="center"
        >
          <template slot-scope="scope">
            <udOperation :data="scope.row" :permission="permission" />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudCust from '@/api/app/cust'
import crudProduct from '@/api/app/product'
import crudSalesdetail from '@/api/app/salesdetail'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
// import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = {
  id: null,
  zhbh: null,
  productid: null,
  businessstatus: null,
  online: null,
  onlinedate: null,
  offlinedate: null,
  onlineleader: null,
  businessman: null,
  reqnum: null,
  contractnum: null,
  constructioncost: null,
  maintenancecost: null,
  note: null,
  createBy: null,
  updateBy: null,
  createTime: null,
  updateTime: null
}
export default {
  name: 'Salesdetail',
  components: { pagination, crudOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['businessStatus'],
  cruds() {
    return CRUD({
      title: '销售情况',
      url: 'api/salesdetail',
      idField: 'id',
      sort: 'id,desc',
      crudMethod: { ...crudSalesdetail }
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'salesdetail:add'],
        edit: ['admin', 'salesdetail:edit'],
        del: ['admin', 'salesdetail:del']
      },
      rules: {
        zhbh: [
          { required: true, message: '租户编号不能为空', trigger: 'blur' }
        ],
        productid: [
          { required: true, message: '产品id不能为空', trigger: 'blur' }
        ],
        businessstatus: [
          { required: true, message: '商务阶段不能为空', trigger: 'blur' }
        ]
      },
      allCust: [], // 所有的客户信息，缓存当做下拉的字典
      allProduct: []// 所有的产品信息
    }
  },
  created() {
    console.log('created')
    this.getAllCust()
    this.getAllProduct()
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    },
    getAllCust: function() {
      crudCust
        .getCusts()
        .then(res => {
          this.allCust = res.content
        })
        .catch(() => {})
    },
    getAllProduct: function() {
      crudProduct.getProducts().then(res => {
        this.allProduct = res.content
      })
        .catch(() => {})
    },
    getCustName: function(zhbh) {
      for (const i in this.allCust) {
        if (this.allCust[i].zhbh === zhbh) {
          return this.allCust[i].khmc
        }
      }
      return zhbh
    },
    getProductName: function(productid) {
      for (const i in this.allProduct) {
        if (this.allProduct[i].id === productid) {
          return this.allProduct[i].productname
        }
      }
      return productid
    }
  }
}
</script>

<style scoped></style>
