<template>
  <div style="width: 100%;">
    <el-form ref="queryForm" :model="queryForm" style="margin-left:20px" inline>
      <el-row>
        <el-form-item label="模板名称" prop="tempName">
          <el-input
            id="tempName"
            v-model="queryForm.tempName"
            plain="true"
            placeholder="请输入计划模板名称"
            clearable
          />
        </el-form-item>
        <el-form-item>
          <el-button icon="el-icon-search" type="primary" class="btn-b" @click="getData(1)">查询</el-button>
          <el-button
            class="btn-w"
            @click="clearSearchBox"
            type="primary"
            icon="el-icon-refresh-left"
          >重置</el-button>
        </el-form-item>
      </el-row>
      <el-row>
        <el-form-item>
          <el-button v-has="'DEV-OPS-LUBRICATION-PLAN-TEMP-ADD'" type="primary" @click="addInspectionTemp()" icon="el-icon-plus">新增</el-button>
          <el-button v-has="'DEV-OPS-LUBRICATION-PLAN-TEMP-BATDELETE'" type="danger" :disabled="batchBtnVisibles" @click="batchDelete">批量删除</el-button>
        </el-form-item>
      </el-row>
    </el-form>
    <el-table stripe border :data="tableData" @selection-change="selsChange" style="width: 100%" height="500px">
      <el-table-column  type="selection" width="55"></el-table-column>
      <el-table-column prop="tempNo" label="模版编码" width="200"></el-table-column>
      <el-table-column prop="tempName" label="模版名称"></el-table-column>
      <el-table-column
        v-if="hasBtn([
          'DEV-OPS-LUBRICATION-PLAN-TEMP-UPDATE',
          'DEV-OPS-LUBRICATION-PLAN-TEMP-DELETE'
        ])"
        align="center"
        label="操作"
        width="230px">
        <template v-slot="scope">
          <el-button v-has="'DEV-OPS-LUBRICATION-PLAN-TEMP-UPDATE'" type="text" size="small" @click="updateplan(scope.row.id)">更新</el-button>
          <el-button v-has="'DEV-OPS-LUBRICATION-PLAN-TEMP-DELETE'" type="text" size="small" @click="deletePlan(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <Pagination
      :total="total"
      :page.sync="page.pageNum"
      :limit.sync="page.pageSize"
      @pagination="getData"
    />
    <el-dialog :title="title" :visible.sync="dialogVisible" width="65%">
      <component
        :is="dialogForm"
        @hidenDialog="hidenDialog"
        :disabled="disabled"
        :tempId="id"
        :datetime="datetime"
      ></component>
    </el-dialog>
  </div>
</template>

<script>
import Pagination from "@/components/Pagination";
import {
  getLubricationTemp,
  deleteLubricationTemp,
  batchDelLubricationTemp
} from "@/api/device";
import lubricationPlanTempAdd from "./lubrication-plan-temp-add";
import lubricationPlanTempUp from "./lubrication-plan-temp-up";
import { hasBtn } from "@/utils/index"

export default {
  name: "CheckingPlanTemplate",
  components: {
    Pagination,
    lubricationPlanTempAdd,
    lubricationPlanTempUp
  },
  data() {
    return {
      page: {
        pageNum: 1,
        pageSize: 10
      },
      total: 0,
      queryForm: {
        tempName: ""
      },
      sels: [], // 选择行数据
      batchBtnVisibles: true,
      tableData: [],
      title: "",
      dialogVisible: false,
      dialogForm: null,
      disabled: null,
      id: "", // 当前行id
      datetime: null
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    hasBtn,
    getData(pageNum) {
      if (pageNum === 1) {
        this.page.pageNum = pageNum;
      }
      const params = {
        ...this.page,
        ...this.queryForm
      };
      getLubricationTemp(params)
        .then(response => {
          const result = response.data;
          if (result.data) {
            this.tableData = result.data.rows;
            this.total = result.data.total;
          }
        })
        .catch(e => {
          this.$message.error(e.message);
        });
    },
    addInspectionTemp() {
      this.dialogVisible = true;
      this.title = "新增";
      this.dialogForm = lubricationPlanTempAdd;
      this.datetime = new Date()
    },
    updateplan(id) {
      this.disabled = false;
      this.dialogVisible = true;
      this.id = id;
      this.title = "更新";
      this.dialogForm = lubricationPlanTempUp;
    },
    deletePlan(id) {
      this.$confirm("此操作将永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          deleteLubricationTemp(id).then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("删除成功");
              this.getData(1);
            } else {
              this.$message.error(result.message);
            }
          });
        })
        .catch(() => {
          this.$message.info("已取消删除");
        });
    },
    hidenDialog() {
      this.getData();
    },
    batchDelete() {
      this.$confirm("此操作将永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          batchDelLubricationTemp(this.sels).then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("删除成功");
              this.getData(1);
            } else {
              this.$message.error(result.message);
            }
          });
        })
        .catch(() => {
          this.$message.info("已取消删除");
        });
    },
    selsChange: function(sels) {
      this.sels.length = 0;
      const _this = this;
      sels.forEach(element => {
        _this.sels.push(element.id);
      });
      this.batchBtnVisibles = this.sels.length === 0;
    },
    clearSearchBox() {
      this.$refs["queryForm"].resetFields();
      this.getData();
    }
  }
};
</script>
