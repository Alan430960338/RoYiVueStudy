<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="回收品类名称" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入回收品类名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="回收品类价格最小值" prop="priceMin">
        <el-input
          v-model="queryParams.priceMin"
          placeholder="请输入回收品类价格最小值"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="回收品类价格最大值" prop="priceMax">
        <el-input
          v-model="queryParams.priceMax"
          placeholder="请输入回收品类价格最大值"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="用友专属商品 code" prop="yonyouCategoryId">
        <el-input
          v-model="queryParams.yonyouCategoryId"
          placeholder="请输入用友专属商品 code"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="好业财商品ID" prop="yonyouProductId">
        <el-input
          v-model="queryParams.yonyouProductId"
          placeholder="请输入好业财商品ID"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="创建人" prop="createUser">
        <el-input
          v-model="queryParams.createUser"
          placeholder="请输入创建人"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="修改人" prop="updateUser">
        <el-input
          v-model="queryParams.updateUser"
          placeholder="请输入修改人"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="是否已删除" prop="deleted">
        <el-input
          v-model="queryParams.deleted"
          placeholder="请输入是否已删除"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="租户ID" prop="tenantId">
        <el-input
          v-model="queryParams.tenantId"
          placeholder="请输入租户ID"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['system:category:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['system:category:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['system:category:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['system:category:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="categoryList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="ID" align="center" prop="id" />
      <el-table-column label="回收品类名称" align="center" prop="name" />
      <el-table-column label="回收品类图片列表" align="center" prop="imageFiles" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.imageFiles" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="回收品类价格最小值" align="center" prop="priceMin" />
      <el-table-column label="回收品类价格最大值" align="center" prop="priceMax" />
      <el-table-column label="用友专属商品 code" align="center" prop="yonyouCategoryId" />
      <el-table-column label="好业财商品ID" align="center" prop="yonyouProductId" />
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="创建人" align="center" prop="createUser" />
      <el-table-column label="修改人" align="center" prop="updateUser" />
      <el-table-column label="是否已删除" align="center" prop="deleted" />
      <el-table-column label="租户ID" align="center" prop="tenantId" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['system:category:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['system:category:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改回收品类对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="categoryRef" :model="form" :rules="rules" label-width="100px">
        <el-row>
          <el-col :span="24">
            <el-form-item label="回收品类名称" prop="name">
              <el-input v-model="form.name" placeholder="请输入回收品类名称" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="回收品类图片列表" prop="imageFiles">
              <el-input v-model="form.imageFiles" type="textarea" placeholder="请输入内容" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="回收品类价格最小值" prop="priceMin">
              <el-input v-model="form.priceMin" placeholder="请输入回收品类价格最小值" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="回收品类价格最大值" prop="priceMax">
              <el-input v-model="form.priceMax" placeholder="请输入回收品类价格最大值" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="用友专属商品 code" prop="yonyouCategoryId">
              <el-input v-model="form.yonyouCategoryId" placeholder="请输入用友专属商品 code" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="好业财商品ID" prop="yonyouProductId">
              <el-input v-model="form.yonyouProductId" placeholder="请输入好业财商品ID" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="备注" prop="remark">
              <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="创建人" prop="createUser">
              <el-input v-model="form.createUser" placeholder="请输入创建人" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="修改人" prop="updateUser">
              <el-input v-model="form.updateUser" placeholder="请输入修改人" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="是否已删除" prop="deleted">
              <el-input v-model="form.deleted" placeholder="请输入是否已删除" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="租户ID" prop="tenantId">
              <el-input v-model="form.tenantId" placeholder="请输入租户ID" />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Category">
import { listCategory, getCategory, delCategory, addCategory, updateCategory } from "@/api/system/category"

const { proxy } = getCurrentInstance()

const categoryList = ref([])
const open = ref(false)
const loading = ref(true)
const showSearch = ref(true)
const ids = ref([])
const single = ref(true)
const multiple = ref(true)
const total = ref(0)
const title = ref("")

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    name: undefined,
    imageFiles: undefined,
    priceMin: undefined,
    priceMax: undefined,
    yonyouCategoryId: undefined,
    yonyouProductId: undefined,
    createUser: undefined,
    updateUser: undefined,
    deleted: undefined,
    tenantId: undefined
  },
  rules: {
    name: [
      { required: true, message: "回收品类名称不能为空", trigger: "blur" }
    ],
    imageFiles: [
      { required: true, message: "回收品类图片列表不能为空", trigger: "blur" }
    ],
    createTime: [
      { required: true, message: "创建时间不能为空", trigger: "blur" }
    ],
    deleted: [
      { required: true, message: "是否已删除不能为空", trigger: "blur" }
    ],
    tenantId: [
      { required: true, message: "租户ID不能为空", trigger: "blur" }
    ]
  }
})

const { queryParams, form, rules } = toRefs(data)

/** 查询回收品类列表 */
function getList() {
  loading.value = true
  listCategory(queryParams.value).then(response => {
    categoryList.value = response.rows
    total.value = response.total
    loading.value = false
  })
}

/** 取消按钮 */
function cancel() {
  open.value = false
  reset()
}

/** 表单重置 */
function reset() {
  form.value = {
    id: null,
    name: null,
    imageFiles: null,
    priceMin: null,
    priceMax: null,
    yonyouCategoryId: null,
    yonyouProductId: null,
    remark: null,
    createUser: null,
    createTime: null,
    updateUser: null,
    updateTime: null,
    deleted: null,
    tenantId: null
  }
  proxy.resetForm("categoryRef")
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1
  getList()
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef")
  handleQuery()
}

/** 多选框选中数据 */
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id)
  single.value = selection.length != 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = "添加回收品类"
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const _id = row.id || ids.value
  getCategory(_id).then(response => {
    form.value = response.data
    open.value = true
    title.value = "修改回收品类"
  })
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["categoryRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateCategory(form.value).then(() => {
          proxy.$modal.msgSuccess("修改成功")
          open.value = false
          getList()
        })
      } else {
        addCategory(form.value).then(() => {
          proxy.$modal.msgSuccess("新增成功")
          open.value = false
          getList()
        })
      }
    }
  })
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value
  proxy.$modal.confirm('是否确认删除回收品类编号为"' + _ids + '"的数据项？').then(function() {
    return delCategory(_ids)
  }).then(() => {
    getList()
    proxy.$modal.msgSuccess("删除成功")
  }).catch(() => {})
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('system/category/export', {
    ...queryParams.value
  }, `category_${new Date().getTime()}.xlsx`)
}

getList()
</script>
