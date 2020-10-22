<template>
  <div>
    <el-dialog
      title="新增语言"
      :visible.sync="dialogVisible"
      width="30%"
      :show-close="false"
    >
      <div class="langBox">
        <el-form ref="form" :model="form" label-width="50px">
          <el-row>
            <el-col :span="12">
              <el-form-item label="语言">
                <el-select v-model="form.langed">
                  <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  >
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="save">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      form: {
        langed: "",
      },
      options: [
        {
          value: "粤语,yue",
          label: "粤语",
        },
        {
          value: "日语,jp",
          label: "日语",
        },
        {
          value: "韩语,kor",
          label: "韩语",
        },
        {
          value: "意大利语,it",
          label: "意大利语",
        },
        {
          value: "法语,fra",
          label: "法语",
        },
      ],
    };
  },
  props: {
    dialogVisible: {
      type: Boolean,
      default: () => false,
    },
  },
  methods: {
    // 保存语言
    save() {
      if (!this.form.langed) {
        this.$message({
          message: "请选择语言进行新增",
          type: "warning",
        });
        return;
      }
      let data = this.form.langed.split(",");
      this.$emit("success", data);
    },
    // 关闭弹框
    close() {
      this.$emit("close");
    },
  },
};
</script>
<style lang="less">
.langBox {
  margin-bottom: 10px;
}
</style>