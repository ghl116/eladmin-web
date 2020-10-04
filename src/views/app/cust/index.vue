<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">客户名称</label>
        <el-input v-model="query.khmc" clearable placeholder="客户名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">客户类别</label>
        <el-select v-model="query.khlb" clearable size="small" placeholder="客户类型" class="filter-item" style="width: 200px" @change="crud.toQuery">
          <el-option v-for="item in dict.khlb" :key="item.id" :value="item.value" :label="item.label" />
        </el-select>
        <rr-operation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crud-operation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="银行编号 " prop="zhbh">
            <el-input v-model="form.zhbh" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="客户名称">
            <el-input v-model="form.khmc" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="客户类别" prop="khlb">
            <el-select v-model="form.khlb" clearable size="small" placeholder="客户类型" class="filter-item" style="width: 370px">
              <el-option v-for="item in dict.khlb" :key="item.id" :value="item.value" :label="item.label" />
            </el-select>
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="form.note" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="zhbh" label="银行编号 " />
        <el-table-column prop="khmc" label="客户名称" />
        <el-table-column prop="khlb" label="客户类别">
          <template slot-scope="scope">

            <span>{{ getKhlbDict(dict.khlb, scope.row.khlb) }}</span>

          </template>
        </el-table-column>
        <el-table-column prop="note" label="备注" />
        <el-table-column prop="createuser" label="创建人" />
        <el-table-column prop="createtime" label="创建时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createtime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="updateuser" label="更新人" />
        <el-table-column prop="updatetime" label="更新时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.updatetime) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','cust:edit','cust:del']" label="操作" width="150px" align="center">
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
import crudCust from '@/api/app/cust'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { zhbh: null, khmc: null, khlb: null, note: null, createuser: null, createtime: null, updateuser: null, updatetime: null }
export default {
  name: 'Cust',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '客户管理cust', url: 'api/cust', idField: 'zhbh', sort: 'updatetime,desc', crudMethod: { ...crudCust }})
  },
  // 设置数据字典
  dicts: ['khlb'],
  data() {
    return {
      permission: {
        add: ['admin', 'cust:add'],
        edit: ['admin', 'cust:edit'],
        del: ['admin', 'cust:del']
      },
      rules: {
        zhbh: [
          { required: true, message: '银行编号 不能为空', trigger: 'blur' }
        ],
        khlb: [
          { required: true, message: '客户类别 不能为空', trigger: 'change' }
        ]
      },
      queryTypeOptions: [
        { key: 'khmc', display_name: '客户名称' },
        { key: 'khlb', display_name: '客户类别' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    },
    getKhlbDict(khlbDict, val) {
      for (const item in khlbDict) {
        if (khlbDict[item].value === val) {
          return khlbDict[item].label
        }
      }
      return val
    }
  }
}
</script>

<style scoped>

</style>
