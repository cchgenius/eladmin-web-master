<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">角色名称</label>
        <el-input v-model="query.roleName" clearable placeholder="角色名称" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <rrOperation :crud="crud"/>
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission"/>
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                 :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="form.roleName" style="width: 370px;"/>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;"
                @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" show-overflow-tooltip align="center"/>
        <el-table-column prop="roleName" label="角色名称" show-overflow-tooltip align="center"/>
        <el-table-column prop="createBy" label="创建者" show-overflow-tooltip align="center"/>
        <el-table-column prop="createTime" label="创建时间" show-overflow-tooltip align="center"/>
        <el-table-column prop="updateBy" label="更新者" show-overflow-tooltip align="center"/>
        <el-table-column prop="updateTime" label="更新时间" show-overflow-tooltip align="center"/>
        <el-table-column v-if="checkPer(['admin','tbRole:edit','tbRole:del'])" label="操作" width="150px"
                         align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination/>
    </div>
  </div>
</template>

<script>
import crudTbRole from '@/api/tbRole'
import CRUD, {presenter, header, form, crud} from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = {roleId: null, roleName: null, createBy: null, createTime: null, updateBy: null, updateTime: null}
export default {
  name: 'TbRole',
  components: {pagination, crudOperation, rrOperation, udOperation},
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({
      title: '角色表',
      url: 'api/tbRole',
      idField: 'roleId',
      sort: 'roleId,desc',
      crudMethod: {...crudTbRole}
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'tbRole:add'],
        edit: ['admin', 'tbRole:edit'],
        del: ['admin', 'tbRole:del']
      },
      rules: {
        roleName: [
          {required: true, message: '角色名称不能为空', trigger: 'blur'}
        ]
      },
      queryTypeOptions: [
        {key: 'roleName', display_name: '角色名称'}
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
