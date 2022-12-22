<template>
    <div class="container">

        <div style="text-align: center;">小程序分析工具</div>
        <el-row :span="24">
            <el-col :xs="0" :sm="0" :md="0" :lg="0" :xl="0">
                <div class="grid-content bg-purple">&nbsp</div>
            </el-col>
            <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="24">
                <div>
                    <el-popover placement="left" width="400" trigger="click" class="popover">
                        <div>
                            <el-input width="30%" placeholder="请匹配内容支持正则表达式" @keyup.enter.native="search" v-model="input" clearable>
                            </el-input>
                            <div style="display:flex;justify-content: space-between;margin-top: 10px;">
                                <el-button @click="search" type="primary" plain style="width:100%" >搜索</el-button>
                                <el-button @click="clear" type="warning" plain > 清空</el-button>
                            </div>
                        </div>

                        <div class="my-search-div" slot="reference">
                            <i class="el-icon-search"></i>
                        </div>
                    </el-popover>

                    <el-dialog title="提示" :show-close="false" :close-on-press-escape="false"
                        :visible.sync="dialogVisible" width="30%">
                        <span><el-input placeholder="请输入WebScoket地址" v-model="WebSocketUrl"></el-input></span>
                        <span slot="footer" class="dialog-footer">
                            <el-button @click="handleClose" type="primary">确 定</el-button>
                        </span>
                    </el-dialog>

                    <el-table :show-header="false" :data="tableDataRet" style="width: 100%">
                        <el-table-column type="expand">
                            <template slot-scope="props">
                                <vueJsonEditor v-model="props.row.resultInfo" :showBtns="false" :mode="'tree'" lang="zh"
                                    :expandedOnStart="false" />
                            </template>
                        </el-table-column>
                        <el-table-column prop="data" :show-overflow-tooltip="true"> </el-table-column>
                    </el-table>

                </div>
            </el-col>
            <el-col :xs="0" :sm="0" :md="0" :lg="0" :xl="0">
                <div class="grid-content bg-purple">&nbsp</div>
            </el-col>
        </el-row>


    </div>
</template>
  
<script>
import vueJsonEditor from "vue-json-editor";
export default {
    components: { vueJsonEditor },
    data() {
        return {
            WebSocketUrl: "",
            dialogVisible: true,
            input: "",
            tableData: [],
            tableDataRet: [],
        };
    },
    mounted() {
        function getQueryString(name) {
            var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
            var r = window.location.search.substr(1).match(reg);
            if (r != null) return unescape(r[2]); return null;
        }
        if (getQueryString('url') != null) {
            this.WebSocketUrl = `ws://${getQueryString('url')}:9090/`
            this.initSocket();
        }
    },
    methods: {
        handleClose() {
            let wsuri = this.WebSocketUrl || "";
            if (wsuri.indexOf("ws://") < 0) {
                wsuri = `ws://${wsuri}`;
            }
            const urlReg =
                /^ws:\/\/(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5]):\d+$/;
            if (!urlReg.test(wsuri)) {
                this.$message.error('websocket地址填写不合法！');
                return;
            }
            this.initSocket();
        },
        search() {
            var that = this;
            that.tableDataRet = []
            for (let i = 0; i < that.tableData.length; i++) {
                var data = that.tableData[i].data;
                if (new RegExp(that.input, "g").test(data)) {
                    that.tableDataRet.push(that.tableData[i])
                }
            }
        },
        clear() {
            this.tableDataRet = []
            this.tableData = []
            this.input = ""
            this.$notify({
                title: '操作!',
                type: 'success',
                message: '清空成功!',
                duration: 5000
            })
        },
        initSocket() {
            if ("WebSocket" in window) {
                this.ws = new WebSocket(this.WebSocketUrl);
                if (this.ws) {
                    this.ws.onopen = (res) => {
                        this.dialogVisible = false
                        this.$notify({
                            title: '操作!',
                            type: 'success',
                            message: 'WebScoket链接成功!',
                            duration: 5000
                        })
                    };
                    let that = this;
                    that.ws.onmessage = (evt) => {
                        let received_msg = evt.data;
                        var jp = {}
                        try {
                            jp = JSON.parse(received_msg)
                        } catch (e) { }

                        jp = formatJson(jp)

                        this.tableData.push({ data: received_msg, resultInfo: jp })
                        if (that.input == "" || new RegExp(that.input, "g").test(received_msg)) {
                            this.tableDataRet.push({ data: received_msg, resultInfo: jp })
                        }
                        function formatJson(jp) {
                            for (var k in jp) {
                                try {
                                    if (jp[k].indexOf('\"') != -1)
                                        jp[k] = formatJson(JSON.parse(jp[k]))
                                } catch (e) {
                                }
                            }
                            return jp
                        }
                    };
                    this.ws.onclose = () => {
                    };
                    this.ws.error = () => {
                    };
                } else {
                }
            }
        },
    },
};
</script>
<style>
.my-search-div {
    width: 72px;
    height: 72px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    /* 圆的直径是 width 和 height 的值，因此需要将 border-radius 设置为 50% */
    background-color: #3F9EFF;
}
.el-icon-search{
    font-size: 36px;
    color:#fff
}


.popover {
    position: fixed;
    right: 50px;
    bottom: 15%;
    z-index: 999;
}

.demo-table-expand {
    font-size: 0;
}

.demo-table-expand label {
    width: 90px;
    color: #99a9bf;
}

.demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
}
</style>