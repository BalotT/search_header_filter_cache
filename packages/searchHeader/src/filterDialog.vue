<template>
    <el-dialog
            title="增加过滤条件"
            :visible.sync="vis"
            :before-close="close"
            width="40%">
        <h4 slot="title" style="margin: 0">增加过滤条件</h4>
        <div class="check-box">
            <div>
                <h4 class="after-class">字段信息</h4>
                <div>
                    <el-checkbox :indeterminate="isIndeterminate"
                                 v-model="checkAll"
                                 @change="handleCheckAllChange">全选
                    </el-checkbox>
                    <div style="margin: 10px 0;"></div>
                    <el-checkbox-group v-model="checkedInput"
                                       @change="handleCheckedInputsChange">
                        <el-checkbox v-for="item in inputList"
                                     :label="item.label"
                                     :key="item.key">{{item.label}}
                        </el-checkbox>
                    </el-checkbox-group>
                </div>
            </div>
            <div>
                <h4 class="after-class check-box-margin">当前所选条件</h4>
                <draggable :list="checkedInput"
                           @start="draggingStart"
                           @end="dragging = false">
                    <transition-group tag="ul" class="ul-class">
                        <li v-for="(item,index) in checkedInput"
                            :style="{'background':dragging&&index===dragIndex?'#c1def9':'#e7f3fd'}"
                            :key="item">
                            <img src="./icon/drag.png" alt="">
                            <span style="cursor: move;color:#2680D1;margin: 0 8px">{{item}}</span>
                            <i style="color: #2680D1" @click='deleteChecked(index)' class="el-icon-close"/>
                        </li>
                    </transition-group>
                </draggable>
            </div>
        </div>
        <div slot="footer" style="text-align: center">
            <el-button type="primary" :size="btnSize" @click="query">确定</el-button>
            <el-button :size="btnSize" @click="close">取消</el-button>
        </div>
    </el-dialog>
</template>

<script>
    import draggable from 'vuedraggable';

    export default {
        name: "filterDialog",
        components: {draggable},
        props: {
            cacheList: {
                type: Array
            },
            vis: {
                type: Boolean
            },
            btnSize: {
                type: String,
                default: 'mini'
            },
            inputList: {
                type: Array,
                default: () => {
                    return []
                }
            },
            name: {
                type: String,
                required: true
            },
        },
        data() {
            return {
                dragging: false,
                dragIndex: 0,
                checkAll: false,
                checkedInput: [],
                isIndeterminate: false,
                // 缓存打开dialog刚打开选中的选项 点击确认时 对比 新勾选的
                // 然后抛给父亲 如果有select就重新请求下拉数据
                cacheCheckedInput:[]
            }
        },
        watch: {
            vis() {
                if (this.vis) {
                    let count = 0;
                    //是否已经有缓存 有缓存半选状态逻辑可以通过缓存数据判断
                    //有缓存 就是有序
                    if (this.cacheList.length > 0) {
                        count = this.cacheList.length;
                        //有可能缓存的输入框里面有的没有了 过滤一次
                        let empty;
                        this.inputList.forEach(item => {
                            empty = this.cacheList.filter(e => {
                                return e === item.label
                            });
                            if (empty && empty.length > 0) {
                                this.checkedInput.push(empty[0])
                            }
                        });
                    } else {
                        this.inputList.forEach(item => {
                            if (item.default) {
                                count++;
                                this.checkedInput.push(item.label);
                            }
                        });
                    }
                    this.isIndeterminate = count > 0 && count < this.inputList.length;
                    this.checkAll = count === this.inputList.length;
                    [...this.cacheCheckedInput] = this.checkedInput;
                }
            }
        },
        methods: {
            draggingStart(e) {
                this.dragging = true;
                this.dragIndex = e.oldIndex;
            },
            handleCheckAllChange(val) {
                if (val) {
                    this.inputList.forEach(item => {
                        this.checkedInput.push(item.label);
                    });
                } else {
                    this.checkedInput = []
                }
                this.checkedInput = Array.from(new Set(this.checkedInput));//去重
                this.isIndeterminate = false;
            },
            handleCheckedInputsChange(value) {
                let checkedCount = value.length;
                this.checkAll = checkedCount === this.inputList.length;
                this.isIndeterminate = checkedCount > 0 && checkedCount < this.inputList.length;
            },
            deleteChecked(index) {
                this.checkedInput.splice(index, 1);
                this.handleCheckedInputsChange(this.checkedInput);
            },
             query() {
                let newLabel= this.checkedInput.filter(item=>{
                       return !this.cacheCheckedInput.includes(item)
                })
                this.$emit('checkedInput', {list:this.checkedInput,newLabel});
                this.close();
            },
            close() {
                this.$emit("update:vis", false);
                this.dragging = false;
                this.dragIndex = 0;
                this.checkAll = false;
                this.checkedInput = [];
                this.isIndeterminate = false;
            },
        }
    }
</script>

<style scoped lang='scss'>
    .check-box {
        padding-left: 15px;

        .after-class {
            position: relative;
            color: #0f76c0;
            margin-bottom: 24px;

            &::before {
                content: "";
                display: block;
                width: 4px;
                height: 14px;
                top: 3px;
                background-color: #0f76c0;
                position: absolute;
                left: -15px;
            }
        }

        .check-box-margin {
            margin-bottom: 14px;
        }

        .ul-class {
            display: flex;
            flex-wrap: wrap;
            padding-left: 0;

            li {
                list-style: none;
                cursor: pointer;
                border: 1px solid #ADD8FF;
                border-radius: 4px;
                margin-right: 8px;
                padding: 3px 8px;
                margin-top: 10px;
                opacity: 0.8;
            }
        }

        /deep/ .el-checkbox {
            margin-bottom: 10px;
            display: inline-flex;
            align-items: center;
            width: 20%;
            margin-right: 5%;

            .el-checkbox__label {
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
        }
    }

    /deep/ .el-dialog__header {
        padding: 24px 24px 0;
    }

    /deep/ .el-dialog__body {
        padding: 0 24px 8px;
    }

    /deep/ .el-button + .el-button {
        margin-left: 8px;
    }

    /deep/ .el-checkbox__label {
        padding-left: 8px;
    }
</style>
