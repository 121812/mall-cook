<!--
 * @Description: 模板商城
 * @Autor: WangYuan
 * @Date: 2021-09-28 17:23:56
 * @LastEditors: WangYuan
 * @LastEditTime: 2021-12-16 16:34:09
-->
<template>
  <div class="wrap">
    <!-- 行业分类 -->
    <ul class="wrap-head">
      <li
        v-for="(item, index) in industryList"
        :key="index"
        class="wrap-head-item"
        :class="[item.value == industry ? 'wrap-head-active' : '']"
        @click="industry = item.value"
      >
        {{ item.label }}
      </li>
    </ul>

    <!-- 模板列表 -->
    <ul v-if="list.length" class="wrap-list">
      <li v-for="model in list" :key="model.id" class="model">
        <template>
          <img :src="model.cover" />
          <div class="model-desc">
            <h3 class="mt5 f14">{{ model.name }}</h3>
            <!-- <div class="mt10 f12 f-grey">设计师：{{ userInfo.userName }}</div> -->
            <el-tag effect="plain" size="mini" class="mt5">{{
              getlIndustryName(model.industry)
            }}</el-tag>
          </div>
        </template>

        <template>
          <div class="model-qr">
            <img class="w90 mb5" :src="getQr(model.id)" />
            <span>扫码预览</span>
          </div>

          <span class="model-btn" @click="useModel(model)">使用模板</span>
        </template>
      </li>
    </ul>

    <!-- 空列表 -->
    <el-empty v-else class="mt80">
      <template slot="description">
        <span class="f13 f-grey">{{ `没有此类型模板哦` }}</span>
      </template>
    </el-empty>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import { getModelList } from "@/api/project";
import { mallIndustryList, mallTypeList } from "@/config/mall";

export default {
  created() {
    this.getlIndustryName();
  },

  data() {
    return {
      industryList: [{ label: "全部", value: "" }, ...mallIndustryList],
      list: [],
      industry: "",
    };
  },

  computed: {
    ...mapGetters(["project", "userInfo"]),
  },

  watch: {
    industry: {
      immediate: true,
      handler() {
        this.getModelList();
      },
    },
  },

  methods: {
    ...mapMutations(["setProject"]),

    async getModelList() {
      let { list } = await getModelList({ industry: this.industry });
      this.list = list;
    },

    getlIndustryName(target) {
      let industryMap = mallIndustryList.reduce((pre, cur) => {
        pre.set(cur.value, cur.label);
        return pre;
      }, new Map());

      return industryMap.get(target);
    },

    useModel(model) {
      let { _id, id, name, userId } = this.project;
      let map = new Map();
      mallTypeList.map((item) => map.set(item.type, item.logo));

      // 模板上配置相关商城数据
      let mall = {
        ...this.$cloneDeep(model),
        ...{ _id, id, name, userId, type: "mall", logo: map.get("mall") },
      };

      this.setProject(mall);
      this.$router.push({ name: "page-build" });

      this.$message({
        type: "success",
        message: "已使用选中模板!",
      });
    },

    getQr(id) {
      let url = `${process.env.VUE_APP_VIEW_API}custom?projectId=${id}`;

      let options = {
        padding: 0, // 二维码四边空白（默认为10px）
        width: 180, // 二维码图片宽度（默认为256px）
        height: 180, // 二维码图片高度（默认为256px）
        correctLevel: QRErrorCorrectLevel.H, // 二维码容错level（默认为高）
        reverse: false, // 反色二维码，二维码颜色为上层容器的背景颜色
        background: "#ffffff", // 二维码背景颜色（默认白色）
        foreground: "#000000", // 二维码颜色（默认黑色）
      };
      console.log("预览地址1:" + url);
      return jrQrcode.getQrBase64(url, options);
    },
  },
};
</script>

<style lang="scss" scoped>
::v-deep .el-button {
  background: $color-theme;
}

.el-tag {
  color: $color-theme;
  border-color: $color-theme;
}

.wrap {
  margin-top: 20px;

  .wrap-head {
    display: flex;
    margin-bottom: 10px;

    .wrap-head-item {
      padding: 8px 12px;
      margin: 0 8px 8px 0;
      background: #f2f2f6;
      border-radius: 4px;
      font-size: 12px;
      text-align: center;
      cursor: pointer;
    }

    .wrap-head-active {
      color: $color-theme;
      background: rgba(37, 137, 255, 0.1);
    }
  }

  .wrap-list {
    height: 700px;
    overflow: auto;

    .model {
      position: relative;
      display: inline-block;
      width: 220px;
      max-height: 380px;
      margin: 0 20px 20px 0;
      border: 1px solid #e9e9e9;
      border-radius: 4px;
      overflow: hidden;
      cursor: pointer;

      img {
        width: 100%;
      }

      .model-desc {
        position: absolute;
        left: 0;
        bottom: -30px;
        width: 100%;
        height: 90px;
        padding: 10px;
        background: hsla(0, 0%, 100%, 0.95);
        font-size: 12px;
        color: #333;
        transition: all 0.3s;
        z-index: 10;
      }

      .model-qr {
        display: none;
        position: absolute;
        top: 20%;
        left: 50%;
        width: 110px;
        padding: 10px;
        border-radius: 6px;
        transform: translateX(-50%);
        background: #fff;
        font-size: 12px;
        text-align: center;
        color: #595961;
        z-index: 10;
      }

      .model-btn {
        display: none;
        position: absolute;
        top: 56%;
        left: 50%;
        width: 110px;
        height: 28px;
        line-height: 28px;
        border-radius: 4px;
        transform: translateX(-50%);
        background: $color-theme;
        font-size: 12px;
        color: #fff;
        text-align: center;
        z-index: 10;
      }

      &:hover {
        &::after {
          content: "";
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: #68656530;
          z-index: 1;
          transition: all 0.3s;
        }

        .model-desc {
          bottom: 0px;
        }

        .model-btn,
        .model-qr {
          display: inline-block;
        }
      }
    }
  }
}
</style>