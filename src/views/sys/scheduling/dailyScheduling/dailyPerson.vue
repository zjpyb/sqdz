<template>
  <!-- 班组排班 -->
  <div class="dailyTeam" style="height:100%;overflow:auto">
    <!-- 查询表单 -->
    <el-form
      :inline="true"
      :rules="rules"
      :model="queryForm"
      class="demo-form-inline"
      style="padding-left:20px"
      ref="queryForm"
    >
      <el-form-item label="排班日期" prop="schedulDate">
        <el-date-picker
          v-model="queryForm.schedulDate"
          type="daterange"
          :picker-options="pickerOptions"
          range-separator="~"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="排班分类" prop="planCategory">
        <el-select
          v-model="queryForm.planCategory"
          placeholder="请选择排班分类"
          @change="getDepartList"
          clearable
        >
          <el-option label="生产排班" value="01"></el-option>
          <el-option label="化验排班" value="02"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="上级组织">
        <el-select v-model="queryForm.parentOrgCode" placeholder="请选择上级组织" clearable>
          <el-option
            v-for="item in parentList"
            :key="item.code"
            :label="item.label"
            :value="item.code"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="scheduling">排班</el-button>
      </el-form-item>
    </el-form>
    <!-- 表格主体 -->
    <el-table :data="tableData" border stripe style="width: 100%" height="calc(100% - 138px)">
      <el-table-column prop="schedulDate" label="日期"></el-table-column>
      <el-table-column prop="schedulPlanCode" label="班制方案"></el-table-column>
      <el-table-column prop="parentOrgName" label="上级组织"></el-table-column>
      <el-table-column prop="shiftName" label="班次"></el-table-column>
      <el-table-column prop="teamName" label="班组"></el-table-column>
      <el-table-column prop="userCode" label="人员" width="200">
        <template v-slot="scope">
          <el-select
            v-model="scope.row.userCode"
            multiple
            collapse-tags
            placeholder="请选择人员"
            @remove-tag="change(scope.row)"
            @change="changePerson(scope.row)"
          >
            <el-option
              v-for="(item,index) in scope.row.userList"
              :key="index"
              :label="item.userName"
              :value="item.userCode"
              @click.native="click(item,scope.row)"
            ></el-option>
          </el-select>
        </template>
      </el-table-column>
    </el-table>
    <!-- 保存行 -->
    <el-row style="padding:20px">
      <el-button type="primary" icon="el-icon-check" @click="save">保存</el-button>
    </el-row>
  </div>
</template>

<script>
import { teamDaily, saveUserDaily, getSltData } from "@/api/sys/scheduling";
import { simpleDateFormat } from "@/utils/index";
import { parse } from "@babel/core";
export default {
  props: {
    activeName: {
      type: String,
      required: false,
      default: "2"
    }
  },
  data() {
    return {
      pickerOptions: {
        // 限制收货时间不让选择今天以前的
        disabledDate(time) {
          return time.getTime() < Date.now() - 24 * 60 * 60 * 1000;
        }
      },
      queryForm: {
        schedulDate: "",
        planCategory: "",
        parentOrgCode: ""
      },
      parentList: [],
      tableData: [],
      list: [],
      rules: {
        schedulDate: [
          {
            required: true,
            message: "请选择日期",
            trigger: ["blur", "change"]
          }
        ],
        planCategory: [
          {
            required: true,
            message: "请选择排班分类",
            trigger: ["blur", "change"]
          }
        ]
      }
    };
  },
  methods: {
    scheduling() {
      this.$refs["queryForm"].validate(val => {
        if (val) {
          let startDate = simpleDateFormat(
            this.queryForm.schedulDate[0],
            "yyyy-MM-dd"
          );
          let endDate = simpleDateFormat(
            this.queryForm.schedulDate[1],
            "yyyy-MM-dd"
          );
          let params = {
            ...this.queryForm,
            startDate,
            endDate,
            schedualType: this.activeName
          };
          delete params.schedulDate;
          teamDaily(params).then(res => {
            if (res.data.success) {
              this.tableData = res.data.data;
              this.tableData = this.tableData.map(item => {
                return {
                  ...item,
                  flag: false
                };
              });
            }
          });
        }
      });
    },
    getDepartList(planCategory) {
      getSltData({ planCategory: planCategory }).then(res => {
        this.parentList = res.data.data;
      });
    },
    click(item, row) {
      if (row.userName.indexOf(item.userName) == "-1") {
        row.userName.push(item.userName);
      } else {
        row.userName = row.userName.filter(v => v != item.userName);
      }
    },
    change(row) {
      row.userName.shift();
    },
    save() {
      let data = JSON.parse(JSON.stringify(this.tableData));
      data = data.filter(item => item.flag);
      data.forEach(item => {
        item.userName = item.userName.join(",");
        item.userCode = item.userCode.join(",");
      });
      saveUserDaily(data).then(res => {
        if (res.data.success) {
          this.$message.success("保存成功");
        } else {
          this.$message.error(res.data.message);
        }
      });
    },
    changePerson(row) {
      row.flag = true;
    }
  },
  watch: {
    "queryForm.planCategory"() {
      this.$set(this.queryForm, "parentOrgCode", "");
    }
  }
};
</script>

<style>
</style>
