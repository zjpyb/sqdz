<template>
  <div class="ene_price_add">
    <el-form :model="tableData" :rules="rules" label-width="180px" ref="tableData">
      <el-col :span="12">
        <el-form-item label="价格名称" prop="name">
          <el-input v-model="tableData.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="能源价格" prop="price">
          <el-input v-model="tableData.price" type="number" autocomplete="off"></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="12" id="enetype">
        <el-form-item label="能源类型" prop="energyCode">
          <el-select
            value="selectValue"
            placeholder="能源类型"
            v-model="tableData.energyCode"
            label="能源类型"
            @change="change"
            style="width: 100%"
          >
            <el-option
              v-for="item in eneType"
              :key="item.code"
              :label="item.label"
              :value="item.code"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="价格单位" prop="unit">
          <el-select
            v-model="tableData.unit"
            value="selectValue"
            placeholder="价格单位"
            style="width: 100%"
          >
            <el-option
              v-for="item in unitPrice"
              :key="item.code"
              :label="item.code"
              :value="item.code"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="生效时间" prop="startTime" style="display:inline-block">
          <el-time-picker v-model="tableData.startTime" placeholder="开始时间"></el-time-picker>
        </el-form-item>
        <el-form-item class="noMargin" style="display:inline-block">~</el-form-item>
        <el-form-item class="noMargin" prop="eneTime" style="display:inline-block">
          <el-time-picker v-model="tableData.endTime" placeholder="结束时间"></el-time-picker>
        </el-form-item>
      </el-col>
    </el-form>

    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click="save('tableData')">保存</el-button>
      <el-button @click="cancel()">取 消</el-button>
      <el-button @click="reset('tableData')">重 置</el-button>
    </div>
  </div>
</template>

<script>
import { addEnePrice, getAllCode, getPriceUnit } from "@/api/energy";
import { simpleDateFormat } from "@/utils/index";

export default {
  name: "enePriceAdd",
  data() {
    const validateCode = (rule, value, callback) => {
      if (this.codes.indexOf(this.tableData.code) > -1) {
        callback(new Error("该能源代码已存在"));
      }
      callback();
    };
    return {
      tableData: {},
      selectValue: "",
      unitPrice: [],
      // timeValue: [simpleDateFormat(new Date(new Date(new Date().toLocaleDateString()).getTime()),"yyyy-MM-dd HH:mm:ss"), simpleDateFormat(new Date(new Date(new Date().toLocaleDateString()).getTime()+24*60*60*1000-1),"yyyy-MM-dd HH:mm:ss")],
      defalutDate: new Date(),
      codes: [],
      rules: {
        name: [
          {
            required: true,
            message: "能源名称必填"
          }
        ],
        code: [
          {
            validator: validateCode,
            trigger: "blur"
          },
          {
            required: true,
            message: "请输入能源代码"
          }
        ],
        energyCode: [
          {
            required: true,
            message: "能源类型必填"
          }
        ],
        price: [
          {
            required: true,
            message: "能源价格必填"
          }
        ],
        unit: [
          {
            required: true,
            message: "能源单位必填"
          }
        ]
      }
    };
  },
  props: ["eneType"],
  mounted() {
    getAllCode()
      .then(res => {
        this.codes = [];
        if (res.data.success) {
          let data = res.data.data;
          data.forEach((item, index, data) => {
            this.codes.push(item.code);
          });
          return;
        }
        this.$message.error(res.data.message);
      })
      .catch(e => {
        this.$message.error(e.message);
      });
  },
  methods: {
    save(tableData) {
      if (this.timeValue === null) {
        this.$message.error("请选择时间范围");
        return;
      }
      this.$refs[tableData].validate(valid => {
        if (valid) {
          this.tableData.startTime = simpleDateFormat(
            this.tableData.startTime,
            "HH:mm:ss"
          );
          this.tableData.endTime = simpleDateFormat(
            this.tableData.endTime,
            "HH:mm:ss"
          );
          const params = {
            ...this.tableData
          };
          addEnePrice(params)
            .then(response => {
              const result = response.data;
              if (result.success) {
                this.$message.success("新增成功");
              } else {
                this.$message.error(result.message);
              }
            })
            .catch(e => {
              this.$message({
                type: "error",
                message: e.message
              });
            })
            .finally(() => {
              this.$refs["tableData"].resetFields();
              this.$emit("hidenDialog");
            });
        } else {
          return false;
        }
      });
    },
    change(value) {
      getPriceUnit(value)
        .then(res => {
          if (res.data.success) {
            this.unitPrice = res.data.data;
            //如果价格存在，那么将unit字段更新
            if (this.unitPrice.length > 0)
              // this.tableData.unit = this.unitPrice[0];
              return;
          }
          this.$message.error(res.data.message);
        })
        .catch(e => {
          this.$message.error(e.message);
        });
    },
    cancel() {
      this.$emit("cancel");
    },
    reset(tableData) {
      this.$refs[tableData].resetFields();
    }
  }
};
</script>

<style scoped>
#enetype {
  height: 51.5px;
}
</style>
<style lang="scss">
.noMargin .el-form-item__content {
  margin-left: 20px !important;
}
</style>
