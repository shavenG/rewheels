<template>
    <div>
        <el-container class="container" id="main">
            <el-container style="position: relative;">
                <el-container>
                    <el-header class="header" height="50px" style="-webkit-app-region: drag">
                        <header-view height="50px" />
                    </el-header>
                    <el-main class="main">
                        <div class="index">
                            <el-tabs v-model="activeName" @tab-click="handleClick">
                                <el-tab-pane label="所有" name="all"></el-tab-pane>
                                <el-tab-pane label="开发类" name="developer"></el-tab-pane>
                                <el-tab-pane label="站长类" name="webmaster"></el-tab-pane>
                                <el-tab-pane label="极客类" name="geek"></el-tab-pane>
                                <el-tab-pane label="维护类" name="devops"></el-tab-pane>
                                <el-tab-pane label="其它" name="other"></el-tab-pane>
                                <el-tab-pane label="奇淫巧技" name="interesting"></el-tab-pane>
                                <el-tab-pane label="网址导航" name="webnavigate"></el-tab-pane>
                            </el-tabs>
                            <div class="index-body">
                                <transition name="el-fade-in-linear">
                                    <keep-alive>
                                        <router-view></router-view>
                                    </keep-alive>
                                </transition>
                            </div>
                        </div>
                    </el-main>
                </el-container>
            </el-container>
        </el-container>
        <el-dialog :visible.sync="dialogDownloadProgressVisible" :show-close="showClose" width="170px">
            <el-progress type="circle" :percentage="downloadPercent"></el-progress>
        </el-dialog>
    </div>
</template>
<script>
    import HeaderView from "../components/Header/";

    export default {
        components: {
            HeaderView
        },
        data() {
            return {
                showClose: false,
                activeName: "all",
                downloadPercent: 0,
                dialogDownloadProgressVisible: false
            };
        },
        methods: {
            handleClick() {
                debugger;
                this.$router.push({
                    path: "/index",
                    query: { activeName: this.activeName }
                });
            }
        },
        created(){
            const ipcRenderer = this.$electron.ipcRenderer;
            ipcRenderer.send("checkForUpdate");

            ipcRenderer.on("message", (event, text) => {
                console.log(arguments);
                this.$message(text);
            });
            //注意：“downloadProgress”事件可能存在无法触发的问题，只需要限制一下下载网速就好了
            ipcRenderer.on("downloadProgress", (event, progressObj)=> {
                console.log(progressObj);
                this.dialogDownloadProgressVisible = true;
                this.downloadPercent = progressObj.percent.toFixed(2) || 0;
            });
            ipcRenderer.on("isUpdateNow", () => {
                // 强制更新
                ipcRenderer.send("isUpdateNow");
            });
        },
        beforeDestroy(){
            this.$electron.ipcRenderer.removeAll(["message", "downloadProgress", "isUpdateNow"]);
        }
    };
</script>
<style lang="scss" scoped>
.aside {
  display: flex;
  flex-direction: column;
  background: linear-gradient(to bottom, #efefef, #efefef);
}

.container {
  height: 100vh;
}

.header {
  background: #fafafa;
}

.main {
  padding: 0;
  margin: 0;
  overflow: hidden;
  display: flex;
  flex: 1;
  flex-shrink: 0;
}

.footer {
  background: #f7f7f7;
  padding: 0;
}

.index {
  display: flex;
  overflow: hidden;
  flex: 1;
  flex-direction: column;
  /deep/ .el-tabs {
    height: 40px;
    background: #fafafa;
    .el-tabs__header {
      padding: 0;
      margin: 0;
    }
    .el-tabs__nav-scroll {
      display: flex;
      justify-content: center;
      padding: 0;
    }
    .el-tabs__nav-wrap::after {
      display: none;
    }
    .el-tabs__item {
      font-size: 14px;
    }
  }
  .index-body {
    background: #f0f0f0;
    flex: 1;
    overflow: auto;
    display: flex;
  }
}
</style>
