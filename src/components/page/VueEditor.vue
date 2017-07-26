<template>
    <div>
        <!-- <quill-editor ref="myTextEditor" v-model="content" :config="editorOption"></quill-editor> -->
        <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="请输入内容" v-model="editContent" @input="textchange" @:onpropertychange="textchange"></el-input>
        <div v-html="content" class="firstContent"></div><el-button class="editor-btn" type="primary" @click="changeData">编辑</el-button>
        <div style="clear:both"></div>
        <div v-html="lastContent"></div>
    </div>
</template>

<script>
    import { quillEditor } from 'vue-quill-editor';
    export default {
        data: function(){
            return {
                content: '第一条、通用服务条款 1) 甲方在购买乙方的招聘产品并在甲方已开通权限范围内，可选择自行操作或委托乙方操作。甲方自行操作的，应及时修改密码，并负责用户名与密码的安全，不得将该用户名及密码提供给第三方',
                editContent: ' ',
                op:{
                    value1: '',
                    value2: ''
                }
                
            }
        },
        components: {
            quillEditor
        },
        computed: {
            lastContent: function (){
                 return this.op.value2;
            }
        },
        methods: {
            changeData(){
                this.editContent = this.content;
                this.lastContent = this.content;
                this.op.value2 = this.editContent;
            },
            textchange() {
                this.op.value1 = this.content;
                this.op.value2 = this.editContent;
                this.eq(this.op);
            },
            eq(op) {
                if(!op){
                    return op;
                }
               /* if(!op.value1_style){
                    op.value1_style="background-color:#FEC8C8;";
                }*/
                if(!op.value2_style){
                    op.value2_style="background-color:#FEC8C8;";
                }
                if(!op.eq_min){
                    op.eq_min=3;
                }
                if(!op.eq_index){
                    op.eq_index=5;
                }
                if (!op.value1 || !op.value2) {
                    return op;
                }
                var ps = {
                    v1_i: 0,
                    v1_new_value: "",
                    v2_i: 0,
                    v2_new_value: ""
                };
                while (ps.v1_i < op.value1.length && ps.v2_i < op.value2.length) {
                    if (op.value1[ps.v1_i] == op.value2[ps.v2_i]) {
                        ps.v1_new_value += op.value1[ps.v1_i].replace(/</g,"<").replace(">",">");
                        ps.v2_new_value += op.value2[ps.v2_i].replace(/</g,"<").replace(">",">");
                        ps.v1_i += 1;
                        ps.v2_i += 1;
                        if (ps.v1_i >= op.value1.length) {
                            ps.v2_new_value += "<span style='" + op.value2_style + "'>" + op.value2.substr(ps.v2_i).replace(/</g,"<").replace(">",">") + "</span>";
                            break;
                        }
                        if (ps.v2_i >= op.value2.length) {
                            ps.v1_new_value += "<span style='" + op.value1_style + "'>" + op.value1.substr(ps.v1_i).replace(/</g,"<").replace(">",">") + "</span>";
                            break;
                        }
                    } else {
                        ps.v1_index = ps.v1_i + 1;
                        ps.v1_eq_length = 0;
                        ps.v1_eq_max = 0;
                        ps.v1_start = ps.v1_i + 1;
                        while (ps.v1_index < op.value1.length) {
                            if (op.value1[ps.v1_index] == op.value2[ps.v2_i + ps.v1_eq_length]) {
                                ps.v1_eq_length += 1;
                            } else if (ps.v1_eq_length > 0) {
                                if (ps.v1_eq_max < ps.v1_eq_length) {
                                    ps.v1_eq_max = ps.v1_eq_length;
                                    ps.v1_start = ps.v1_index - ps.v1_eq_length;
                                }
                                ps.v1_eq_length = 0;
                                break;//只寻找最近的
                            }
                            ps.v1_index += 1;
                        }
                        if (ps.v1_eq_max < ps.v1_eq_length) {
                            ps.v1_eq_max = ps.v1_eq_length;
                            ps.v1_start = ps.v1_index - ps.v1_eq_length;
                        }

                        ps.v2_index = ps.v2_i + 1;
                        ps.v2_eq_length = 0;
                        ps.v2_eq_max = 0;
                        ps.v2_start = ps.v2_i + 1;
                        while (ps.v2_index < op.value2.length) {
                            if (op.value2[ps.v2_index] == op.value1[ps.v1_i + ps.v2_eq_length]) {
                                ps.v2_eq_length += 1;
                            } else if (ps.v2_eq_length > 0) {
                                if (ps.v2_eq_max < ps.v2_eq_length) {
                                    ps.v2_eq_max = ps.v2_eq_length;
                                    ps.v2_start = ps.v2_index - ps.v2_eq_length;
                                }
                                ps.v1_eq_length = 0;
                                break;//只寻找最近的
                            }
                            ps.v2_index += 1;
                        }
                        if (ps.v2_eq_max < ps.v2_eq_length) {
                            ps.v2_eq_max = ps.v2_eq_length;
                            ps.v2_start = ps.v2_index - ps.v2_eq_length;
                        }
                        if (ps.v1_eq_max < op.eq_min && ps.v1_start - ps.v1_i > op.eq_index) {
                            ps.v1_eq_max = 0;
                        }
                        if (ps.v2_eq_max < op.eq_min && ps.v2_start - ps.v2_i > op.eq_index) {
                            ps.v2_eq_max = 0;
                        }
                        if ((ps.v1_eq_max == 0 && ps.v2_eq_max == 0)) {
                            ps.v1_new_value += "<span style='" + op.value1_style + "'>" + op.value1[ps.v1_i].replace(/</g,"<").replace(">",">") + "</span>";
                            ps.v2_new_value += "<span style='" + op.value2_style + "'>" + op.value2[ps.v2_i].replace(/</g,"<").replace(">",">") + "</span>";
                            ps.v1_i += 1;
                            ps.v2_i += 1;

                            if (ps.v1_i >= op.value1.length) {
                                ps.v2_new_value += "<span style='" + op.value2_style + "'>" + op.value2.substr(ps.v2_i).replace(/</g,"<").replace(">",">") + "</span>";
                                break;
                            }
                            if (ps.v2_i >= op.value2.length) {
                                ps.v1_new_value += "<span style='" + op.value1_style + "'>" + op.value1.substr(ps.v1_i).replace(/</g,"<").replace(">",">") + "</span>";
                                break;
                            }
                        } else if (ps.v1_eq_max > ps.v2_eq_max) {
                            ps.v1_new_value += "<span style='" + op.value1_style + "'>" + op.value1.substr(ps.v1_i, ps.v1_start - ps.v1_i).replace(/</g,"<").replace(">",">") + "</span>";
                            ps.v1_i = ps.v1_start;
                        } else {
                            ps.v2_new_value += "<span style='" + op.value2_style + "'>" + op.value2.substr(ps.v2_i, ps.v2_start - ps.v2_i).replace(/</g,"<").replace(">",">") + "</span>";
                            ps.v2_i = ps.v2_start;
                        }
                    }
                }
                op.value1 = ps.v1_new_value;
                op.value2 = ps.v2_new_value;
                return op;
            }
        }
    }
</script>
<style scoped>
    .firstContent{
        width: 200px;
        float: left;
        padding: 5px;
        border: 1px solid #000;
        margin: 20px;
    }
    .editor-btn{
        margin-top: 20px;
        float: left;
    }
</style>