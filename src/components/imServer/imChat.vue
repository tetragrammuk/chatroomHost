<!-- 聊天记录 -->
<template>
  <div class="imChat-wrapper">
    <!-- 头部 -->
    <header class="imChat-header">
      <span class="name">{{storeSelectedChatEn.clientChatName}}</span>
      <span class="time">{{getAccessTimeStr(storeSelectedChatEn.accessTime)}}</span>
      <span v-show="storeSelectedChatEn.state=='on' " class="on-line">在線</span>
      <span v-show="storeSelectedChatEn.state=='off' " class="off-line">離線</span>
      <div class="button">
        <el-button
          v-if="storeCurrentChatEnlist_done.map(function(item, index) {
                        return item.clientChatId;
                    }).indexOf(storeSelectedChatEn.clientChatId)==-1"
          class="done"
          type="primary"
          @click="done_change"
        >移到已處理</el-button>
        <el-button
          v-if="storeCurrentChatEnlist_done.map(function(item, index) {
                        return item.clientChatId;
                    }).indexOf(storeSelectedChatEn.clientChatId)!=-1"
          class="done"
          type="primary"
          @click="non_done_change"
        >移到未處理</el-button>
      </div>
    </header>
    <main class="imChat-main">
      <!-- 聊天框区域 -->
      <common-chat
        ref="common_chat"
        :chatInfoEn="storeSelectedChatEn"
        :oprRoleName="'server'"
        @sendMsg="sendMsg"
      ></common-chat>
    </main>
  </div>
</template>

<script>
import commonChat from "@/components/common/common_chat.vue";
import axios from "axios";
export default {
  components: {
    commonChat: commonChat
  },
  data() {
    return {};
  },
  computed: {
    storeCurrentChatEnlist: {
      get() {
        return this.$store.imServerStore.state.currentChatEnlist;
      },
      set(data) {
        this.$store.imServerStore.commit("UPDATE_CHATLIST", data);
      }
    },
    storeCurrentChatEnlist_done: {
      get() {
        return this.$store.imServerStore.state.currentChatEnlist_done;
      },
      set(data) {
        this.$store.imServerStore.commit("UPDATE_CHATLIST_DONE", data);
      }
    },

    storeSelectedChatEn() {
      return this.$store.imServerStore.getters.selectedChatEn;
    },
    storeHaveNewMsgDelegate() {
      return this.$store.imServerStore.getters.haveNewMsgDelegate;
    },
    storeServerChatEn() {
      return this.$store.imServerStore.getters.serverChatEn;
    }
  },
  watch: {
    storeSelectedChatEn(value) {
      this.$refs.common_chat.goEnd();
    },
    storeHaveNewMsgDelegate(value) {
      this.$refs.common_chat.goEnd();
    }
  },
  methods: {
    done_change: function() {
      this.$store.imServerStore.commit("done_change", {
        storeSelectedChatEn: this.storeSelectedChatEn
      });
      axios({
        method: "post",
        url: "http://127.0.0.1:3000/api/ChatEn_update",
        data: {
          serverChatId: this.storeServerChatEn.serverChatId,
          ChatEnList: this.storeCurrentChatEnlist,
          done_ChatEnList: this.storeCurrentChatEnlist_done
        },
        headers: {
          "Content-Type": "application/json"
        }
      }).then(response => {
        console.log(response);
      });
    },

    non_done_change: function() {
      this.$store.imServerStore.commit("non_done_change", {
        storeSelectedChatEn: this.storeSelectedChatEn
      });
      axios({
        method: "post",
        url: "http://127.0.0.1:3000/api/ChatEn_update",
        data: {
          serverChatId: this.storeServerChatEn.serverChatId,
          ChatEnList: this.storeCurrentChatEnlist,
          done_ChatEnList: this.storeCurrentChatEnlist_done
        },
        headers: {
          "Content-Type": "application/json"
        }
      }).then(response => {
        console.log(response);
      });
    },
    /**
     * 发送消息
     * @param {Object} rs 回调对象
     */
    sendMsg: function(rs) {
      var msg = rs.msg;
      msg.role = "server";
      msg.avatarUrl = this.storeServerChatEn.avatarUrl;
      // 1.socket发送消息
      this.$store.imServerStore.dispatch("sendMsg", {
        clientChatId: this.storeSelectedChatEn.clientChatId,
        msg: msg
      });

      // 2.附加到此chat对象的msg集合里
      this.$store.imServerStore.dispatch("addChatMsg", {
        clientChatId: this.storeSelectedChatEn.clientChatId,
        msg: msg,
        successCallback: function() {
          rs.successCallbcak && rs.successCallbcak();
        }
      });
    },
    goEnd: function() {
      this.$refs.common_chat.goEnd();
    },

    /**
     * 获取chat的访问时间
     * @param {Date} accessTime 问时间
     */
    getAccessTimeStr: function(accessTime) {
      return this.$ak.Utils.getDateTimeStr(accessTime, "Y-m-d H:i:s");
    }
  },
  mounted() {}
};
</script>
<style lang="less">
.imChat-wrapper {
  .imChat-header {
    display: flex;
    align-items: center;
    width: 100%;
    height: 50px;
    padding-left: 10px;
    border-bottom: 1px solid #e6e6e6;
    font-size: 16px;
    span {
      margin-right: 20px;
    }
    .on-line {
      color: #70ed3a;
    }
    .off-line {
      color: #bbbbbb;
    }
    .button {
      margin-right: 0px;
    }
  }
  .imChat-main {
    height: calc(~"100% - 50px");
  }
}
</style>

