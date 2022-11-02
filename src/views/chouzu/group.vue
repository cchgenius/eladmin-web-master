<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">分组名称</label>
        <el-input v-model="query.groupName" clearable placeholder="分组名称" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <!--        <label class="el-form-item-label">分组方式</label>
                <el-input v-model="query.groupWay" clearable placeholder="分组方式" style="width: 185px;" class="filter-item"
                          @keyup.enter.native="crud.toQuery"/>-->
        <rrOperation :crud="crud"/>
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission"/>
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                 :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="分组名称" prop="groupName">
            <el-input v-model="form.groupName" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="分组方式" prop="groupWay">
            <el-radio v-model="form.groupWay" v-for="item in dict.group_way" :key="item.id" :label="item.value">
              {{ item.label }}
            </el-radio>
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
        <el-table-column prop="groupName" label="分组名称" show-overflow-tooltip align="center"/>
        <el-table-column prop="groupWay" label="分组方式" show-overflow-tooltip align="center">
          <template slot-scope="scope">
            {{ dict.label.group_way[scope.row.groupWay] }}
          </template>
        </el-table-column>
        <el-table-column prop="createBy" label="创建者" show-overflow-tooltip align="center"/>
        <el-table-column prop="createTime" label="创建时间" show-overflow-tooltip align="center"/>
        <el-table-column prop="updateBy" label="更新者" show-overflow-tooltip align="center"/>
        <el-table-column prop="updateTime" label="更新时间" show-overflow-tooltip align="center"/>
        <el-table-column v-if="checkPer(['admin','tbGroup:edit','tbGroup:del'])" label="操作" width="150px"
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
import crudTbGroup from '@/api/tbGroup'
import CRUD, {presenter, header, form, crud} from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = {
  gruopId: null,
  groupName: null,
  groupWay: null,
  createBy: null,
  createTime: null,
  updateBy: null,
  updateTime: null
}
export default {
  name: 'TbGroup',
  components: {pagination, crudOperation, rrOperation, udOperation},
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['group_way'],
  cruds() {
    return CRUD({
      title: '分组表',
      url: 'api/tbGroup',
      idField: 'gruopId',
      sort: 'gruopId,desc',
      crudMethod: {...crudTbGroup}
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'tbGroup:add'],
        edit: ['admin', 'tbGroup:edit'],
        del: ['admin', 'tbGroup:del']
      },
      rules: {
        groupName: [
          {required: true, message: '分组名称不能为空', trigger: 'blur'}
        ],
        groupWay: [
          {required: true, message: '分组方式不能为空', trigger: 'blur'}
        ]
      },
      queryTypeOptions: [
        {key: 'groupName', display_name: '分组名称'},
        {key: 'groupWay', display_name: '分组方式'}
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
