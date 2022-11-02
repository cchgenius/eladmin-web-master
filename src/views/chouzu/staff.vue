<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">人员名称</label>
        <el-input v-model="query.name" clearable placeholder="人员名称" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">手机号码</label>
        <el-input v-model="query.phone" clearable placeholder="手机号码" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">邮箱</label>
        <el-input v-model="query.email" clearable placeholder="邮箱" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">是否已分组</label>
        <el-input v-model="query.isGruop" clearable placeholder="是否已分组" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <rrOperation :crud="crud"/>
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission"/>
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                 :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="人员名称" prop="name">
            <el-input v-model="form.name" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="性别" prop="gender">
            <el-radio v-for="item in dict.gender" :key="item.id" v-model="form.gender" :label="item.label">
              {{ item.label }}
            </el-radio>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model="form.age" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="手机号码" prop="phone">
            <el-input v-model="form.phone" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="form.email" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="角色" prop="roleName">
            <template slot-scope="scope">
              <el-select v-model="form.roleName">
                <el-option v-for="item in dict.role_name" :key="item.id" label="item.value">
                  {{ item.label }}
                </el-option>
              </el-select>
            </template>
            <!--            <el-input v-model="form.email" style="width: 370px;"/>-->
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
        <el-table-column prop="name" label="人员名称" show-overflow-tooltip align="center"/>
        <el-table-column prop="gender" label="性别" show-overflow-tooltip align="center">
          <template slot-scope="scope">
            {{ scope.row.gender }}
          </template>
        </el-table-column>
        <el-table-column prop="age" label="年龄" show-overflow-tooltip align="center"/>
        <el-table-column prop="phone" label="手机号码" show-overflow-tooltip align="center"/>
        <el-table-column prop="email" label="邮箱" show-overflow-tooltip align="center"/>
        <el-table-column prop="isGruop" label="是否已分组" show-overflow-tooltip align="center"/>
        <el-table-column prop="gruopId" label="分组名称" show-overflow-tooltip align="center"/>
        <!--        <el-table-column prop="createBy" label="创建者" show-overflow-tooltip align="center"/>
                <el-table-column prop="createTime" label="创建时间" show-overflow-tooltip align="center"/>
                <el-table-column prop="updateBy" label="更新者" show-overflow-tooltip align="center"/>
                <el-table-column prop="updateTime" label="更新时间" show-overflow-tooltip align="center"/>-->
        <el-table-column prop="roleId" label="角色" show-overflow-tooltip align="center">
          <template slot-scope="scope">
            {{ dict.label.role_name[scope.row.roleId] }}
          </template>
        </el-table-column>
        <el-table-column v-if="checkPer(['admin','tbStaff:edit','tbStaff:del'])" label="操作" width="150px"
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
import crudTbStaff from '@/api/tbStaff'
import CRUD, {presenter, header, form, crud} from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = {
  staffId: null,
  gruopId: null,
  name: null,
  gender: null,
  age: null,
  phone: null,
  email: null,
  isGruop: null,
  createBy: null,
  createTime: null,
  updateBy: null,
  updateTime: null,
  roleId: null,
  roleName: null
}
export default {
  name: 'TbStaff',
  components: {pagination, crudOperation, rrOperation, udOperation},
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['gender', 'role_name'],
  cruds() {
    return CRUD({
      title: '人员',
      url: 'api/tbStaff',
      idField: 'staffId',
      sort: 'staffId,desc',
      crudMethod: {...crudTbStaff}
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'tbStaff:add'],
        edit: ['admin', 'tbStaff:edit'],
        del: ['admin', 'tbStaff:del']
      },
      rules: {
        name: [
          {required: true, message: '分组名称不能为空', trigger: 'blur'}
        ],
        gender: [
          {required: true, message: '分组方式不能为空', trigger: 'blur'}
        ],
        age: [
          {required: true, message: '分组名称不能为空', trigger: 'blur'}
        ],
        phone: [
          {required: true, message: '分组方式不能为空', trigger: 'blur'}
        ],
        email: [
          {required: true, message: '分组名称不能为空', trigger: 'blur'}
        ],
        roleName: [
          {required: true, message: '请选择角色', trigger: 'change'}
        ]
      },
      queryTypeOptions: [
        {key: 'name', display_name: '人员名称'},
        {key: 'phone', display_name: '手机号码'},
        {key: 'email', display_name: '邮箱'},
        {key: 'isGruop', display_name: '是否已分组'}
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
