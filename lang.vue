<template>
  <div>
    <div>
      <el-button type="primary" @click="addRow">新增行</el-button>
      <el-button type="primary" @click="addLang">新增语言</el-button>
      <el-button type="danger" @click="deleteRow">删除</el-button>
    </div>
    <div class="tableBox">
      <el-table
        :data="langTable"
        border
        @selection-change="handleSelectionChange"
        ref="table"
        v-if="tableShow"
      >
        <el-table-column type="selection" label="序号" width="55">
        </el-table-column>
        <el-table-column
          v-for="(table, tabIndex) in column"
          :key="tabIndex"
          :prop="table.prop"
          :label="table.label"
          :width="table.width"
        >
          <template slot-scope="scope">
            <div class="input-row">
              <el-input
                v-model="scope.row[table.prop]"
                v-if="table.prop === 'key'"
              ></el-input>
              <el-input
                v-model="scope.row[table.prop]"
                v-else
                @blur="
                  lang(
                    scope.$index,
                    scope.row[table.prop],
                    table.prop,
                    scope.row
                  )
                "
              ></el-input>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <addColumn
      :dialogVisible="dialogVisible"
      @success="addSuccess"
      @close="addClose"
    ></addColumn>
  </div>
</template>
<script>
import md5 from "@/utils/md5";
import addColumn from "./addColumn";
export default {
  data() {
    return {
      langTable: [], // 表格数据
      column: [
        {
          prop: "key",
          label: "key",
          width: "",
        },
        {
          prop: "zh",
          label: "中文",
          width: "",
        },
        {
          prop: "en",
          label: "英语",
          width: "",
        },
      ], // 默认表格列
      befor: {
        q: "",
      },
      q: "", //输入的原文
      to: "", //选择目标语言
      appid: "20201015000590226", //百度翻译开放平台的个人AppId
      salt: "21376874", //随机数
      from: "auto", //源语言
      sign: "", //签名
      userkey: "PgRw7im8rWfCz4Q4ZhZ9", //百度翻译开放平台的个人密匙
      dialogVisible: false, // 是否打开新增语言弹框
      selectArray: [], // 表格中选中记录在数组里
      tableShow: true,
    };
  },
  components: { addColumn },
  methods: {
    // 新增行
    addRow() {
      // this.langTable.push({ key: "", zh: "", en: "" });
      let arr = [];
      let obj = {};
      arr = this.column.map((item) => {
        obj[item.prop] = "";
        return obj;
      });
      this.langTable.push(arr[0]);
      // this.langTable.push(Object.assign({}, arr));
      // let obj = arr.map(m=>{

      // })
    },
    // 新增语言
    addLang() {
      this.dialogVisible = true;
    },
    // 新增成功后把新增语言添加到表格中
    addSuccess(data) {
      this.dialogVisible = false;
      this.column.push({ label: data[0], prop: data[1], width: "" });
      if (this.langTable.length > 0) {
        this.langTable.forEach((item, idx) => {
          Object.keys(item).forEach((o) => {
            if (o !== data[1]) {
              item[data[1]] = "";
            }
          });
          // 添加后自动编译
          if (item["zh"]) {
            this.lang(idx, item["zh"], "", item);
          }
        });
      }
    },
    // 关闭弹框
    addClose() {
      this.dialogVisible = false;
    },
    // 选择多选框
    handleSelectionChange(selection) {
      this.selectArray = selection;
    },
    // 删除选中的行
    deleteRow() {
      this.selectArray.forEach((item) => {
        // 如果还没输入key的话 先暂时不删
        if (item.key) {
          this.langTable = this.langTable.filter((r) => r.key !== item.key);
        }
      });
    },
    // 翻译语言
    lang(tabIndex, text, prop, scope) {
      // 如果没有输入翻译内容跳出请求
      if (!text) return;
      /* 待翻译文本 传入url */
      this.befor.q = text;
      this.q = this.befor.q;
      /* 从页面获取选择的目标语言 传入url */
      this.to = this.getLang(prop, scope);
      /* md5加密，生成签名 */
      this.sign = md5(
        this.appid + this.toUtf8(this.befor.q) + this.salt + this.userkey
      );
      /* 对待翻译字符做url编码 */
      this.q = encodeURIComponent(this.q);
      // 如果有多语言就循环调接口
      this.to.forEach((item) => {
        /* 请求翻译 */
        this.$http
          .get(
            "/api/trans/vip/translate" +
              "?q=" +
              this.q +
              "&from=" +
              this.from +
              "&to=" +
              item +
              "&appid=" +
              this.appid +
              "&salt=" +
              this.salt +
              "&sign=" +
              this.sign
          )
          /* 得到返回数据 */
          .then((res) => {
            this.tableShow = false;
            this.$nextTick(() => {
              this.$set(
                this.langTable[tabIndex],
                item,
                res.data.trans_result[0].dst
              );
              this.tableShow = true;
            });
            this.tableShow = true;
            // scope[item] = res.data.trans_result[0].dst; //得到翻译结果
          })
          .catch((err) => {
            console.log(err);
            this.tableShow = true;
          });
      });
    },
    // 根据输入的语言找出其它语言
    getLang(prop, data) {
      console.log(data);
      let res = [];
      Object.keys(data).forEach((item) => {
        if (item !== prop && item !== "key" && !data[item]) {
          res.push(item);
        }
      });
      return res;
    },
    // 把翻译的内容转换为utf-8格式
    toUtf8(str) {
      var out, i, len, c;
      out = "";
      len = str.length;
      for (i = 0; i < len; i++) {
        c = str.charCodeAt(i);
        if (c >= 0x0001 && c <= 0x007f) {
          out += str.charAt(i);
        } else if (c > 0x07ff) {
          out += String.fromCharCode(0xe0 | ((c >> 12) & 0x0f));
          out += String.fromCharCode(0x80 | ((c >> 6) & 0x3f));
          out += String.fromCharCode(0x80 | ((c >> 0) & 0x3f));
        } else {
          out += String.fromCharCode(0xc0 | ((c >> 6) & 0x1f));
          out += String.fromCharCode(0x80 | ((c >> 0) & 0x3f));
        }
      }
      return out;
    },
  },
};
</script>
<style lang="less">
.tableBox {
  margin-top: 10px;
}
.input-row {
  margin-right: 20px;
}
</style>