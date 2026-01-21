<template>
  <div class="app-container">
    <el-form
      width="600"
      :model="queryParams"
      ref="queryRef"
      :inline="true"
      v-show="showSearch"
      label-width="68px"
    >
      <el-form-item label="标题" prop="title">
        <el-input
          v-model="queryParams.title"
          placeholder="请输入标题"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="分类" prop="categoryId">
        <el-select
          v-model="queryParams.categoryId"
          placeholder="请选择分类"
          clearable
        >
          <el-option
            v-for="dict in blog_article_category"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="标签" prop="tags">
        <el-input
          v-model="queryParams.tags"
          placeholder="请输入标签"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="请选择状态"
          clearable
        >
          <el-option
            v-for="dict in blog_article_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="创建人" prop="createBy">
        <el-input
          v-model="queryParams.createBy"
          placeholder="创建人"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery"
          >搜索</el-button
        >
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
          v-hasPermi="['blog:article:add']"
          >新增</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['blog:article:edit']"
          >修改</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['blog:article:remove']"
          >删除</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['blog:article:export']"
          >导出</el-button
        >
      </el-col>
      <right-toolbar
        v-model:showSearch="showSearch"
        @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="articleList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        type="selection"
        fixed="left"
        width="55"
        align="center"
      />
      <el-table-column label="ID" fixed="left" align="center" prop="id" />
      <el-table-column label="标题" fixed="left" align="center" prop="title" />
      <el-table-column
        width="220"
        show-overflow-tooltip
        label="内容"
        align="center"
        prop="content"
      />
      <el-table-column width="120" label="摘要" align="center" prop="summary" />
      <el-table-column
        label="分类"
        width="120"
        align="center"
        prop="categoryId"
      >
        <template #default="scope">
          <dict-tag
            :options="blog_article_category"
            :value="scope.row.categoryId"
          />
        </template>
      </el-table-column>
      <el-table-column width="120" label="标签" align="center" prop="tags" />
      <el-table-column label="状态" align="center" prop="status">
        <template #default="scope">
          <dict-tag :options="blog_article_status" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column label="浏览量" align="center" prop="viewCount" />
      <el-table-column label="点赞量" align="center" prop="likeCount" />
      <el-table-column label="评论量" align="center" prop="commentCount" />
      <el-table-column label="创建人" align="center" prop="createBy" />
      <el-table-column label="更新人" align="center" prop="updateBy" />
      <el-table-column
        width="220"
        label="创建时间"
        align="center"
        prop="createTime"
      />
      <el-table-column
        width="220"
        label="更新时间"
        align="center"
        prop="updateTime"
      />
      <el-table-column
        width="220"
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
        fixed="right"
      >
        <template #default="scope">
          <el-button
            link
            type="primary"
            icon="Edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['blog:article:edit']"
            >修改</el-button
          >
          <el-button
            link
            type="primary"
            icon="Delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['blog:article:remove']"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改文章对话框 -->
    <el-dialog :title="title" v-model="open" width="80%" append-to-body>
      <el-form ref="articleRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入标题" />
        </el-form-item>
        <el-form-item label="摘要" prop="summary">
          <el-input
            v-model="form.summary"
            type="textarea"
            placeholder="请输入内容"
          />
        </el-form-item>
        <el-form-item label="分类" prop="categoryId">
          <el-select v-model="form.categoryId" placeholder="请选择分类">
            <el-option
              v-for="dict in blog_article_category"
              :key="dict.value"
              :label="dict.label"
              :value="+dict.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="标签" prop="tags">
          <el-input v-model="form.tags" placeholder="请输入标签" />
        </el-form-item>
        <el-form-item label="内容" prop="content">
          <!-- <editor v-model="form.content" :min-height="192" />-->
          <md-editor
            v-model="form.content"
            @onUploadImg="onUploadImg"
            :theme="isDark ? 'dark' : 'light'"
          />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="form.status" placeholder="请选择状态">
            <el-option
              v-for="dict in blog_article_status"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            ></el-option>
          </el-select>
        </el-form-item>
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

<script setup name="Article">
import { MdEditor } from "md-editor-v3";
import "md-editor-v3/lib/style.css";
import {
  listArticle,
  getArticle,
  delArticle,
  addArticle,
  updateArticle,
} from "@/api/blog/article";
import { status } from "nprogress";

const { proxy } = getCurrentInstance();
const { blog_article_status, blog_article_category } = proxy.useDict(
  "blog_article_status",
  "blog_article_category",
);

const articleList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");
const isDark = ref(false);

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    title: null,
    content: null,
    summary: null,
    categoryId: null,
    tags: null,
    status: null,
    createBy: null,
  },
  rules: {
    title: [{ required: true, message: "标题不能为空", trigger: "blur" }],
    summary: [{ required: true, message: "摘要不能为空", trigger: "blur" }],
    categoryId: [
      { required: true, message: "分类不能为空", trigger: "change" },
    ],
    content: [{ required: true, message: "博客内容不能为空", trigger: "blur" }],
    tags: [{ required: true, message: "标签不能为空", trigger: "blur" }],
    status: [{ required: true, message: "状态不能为空", trigger: "change" }],
  },
});

const { queryParams, form, rules } = toRefs(data);

/** 查询文章列表 */
function getList() {
  loading.value = true;
  listArticle(queryParams.value).then((response) => {
    articleList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    title: null,
    content: "",
    summary: null,
    categoryId: null,
    tags: null,
    status: null,
    createTime: null,
    updateTime: null,
  };
  proxy.resetForm("articleRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map((item) => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加文章";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value;
  getArticle(_id).then((response) => {
    form.value = response.data;
    open.value = true;
    title.value = "修改文章";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["articleRef"].validate((valid) => {
    if (valid) {
      if (form.value.id != null) {
        updateArticle(form.value).then((response) => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addArticle(form.value).then((response) => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal
    .confirm('是否确认删除文章编号为"' + _ids + '"的数据项？')
    .then(function () {
      return delArticle(_ids);
    })
    .then(() => {
      getList();
      proxy.$modal.msgSuccess("删除成功");
    })
    .catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download(
    "blog/article/export",
    {
      ...queryParams.value,
    },
    `article_${new Date().getTime()}.xlsx`,
  );
}

// 图片上传处理
const onUploadImg = async (files, callback) => {
  const res = await Promise.all(
    files.map((file) => {
      return new Promise((rev, rej) => {
        const form = new FormData();
        form.append("file", file);
        // 调用若依默认的上传接口
        uploadFile(form)
          .then((res) => rev(res))
          .catch((error) => rej(error));
      });
    }),
  );

  // 回显图片地址到编辑器
  callback(res.map((item) => item.url));
};

getList();
</script>
