<template>
    <el-container class="path-bar">
        <el-header style="display:flex;height:35px;line-height:35px;padding:0px;">
            <el-button type="text" :icon="controlStatus" @click="control.show=!control.show"></el-button>
            <template v-if="control.show">
                <div style="width:84%;">
                    <el-radio-group v-model="entity.path.type">
                        <el-radio-button label="all">全路径</el-radio-button>
                        <el-radio-button label="short">最短路径</el-radio-button>
                        <el-radio-button label="long">最长路径</el-radio-button>
                        <el-radio-button label="">关键路径</el-radio-button>
                    </el-radio-group>
                </div>
                <div>
                    <el-divider direction="vertical"></el-divider>
                </div>
                <el-button type="text" @click="$emit('toggle-view','SearchBar')">
                    <span class="el-icon-close" style="font-size:14px;font-weight: 900;"></span>
                </el-button>
                <el-tooltip content="路劲查询" placement="top">
                    <el-button type="success" class="search" :loading="entity.search.loading">
                        <span class="el-icon-search"></span>
                    </el-button>
                </el-tooltip>
                
            </template>
        </el-header>
        <el-main class="path-main" v-show="control.show">
            <PathView class="graphAction" 
                :model="model" 
                :pathType="entity.path.type" 
                ref="pathRef"></PathView>
        </el-main>
    </el-container>
</template>

<script>

import _ from 'lodash';
import toggleBarMixin from '../../mixins/index.js';
import PathView from './path/index';

export default {
    name: "PathBar",
    mixins: [toggleBarMixin],
    components:{
        PathView
    },
    data(){
        return {
            model: null,
            control: {
                show: true
            },
            entity: {
                search: {
                    loading: false,
                    term: "",
                    result: null
                },
                path: {
                    type: "all"
                },
                selected: null
            }
        }
    },
    watch: {
        'entity.search.term':function(val){
            if(_.isEmpty(val)){
                this.onEntityClear();
            }
        }
    },
    computed:{
        controlStatus(){
            if(this.control.show){
                return 'el-icon-arrow-left';
            } else {
                return 'el-icon-arrow-right';
            }
        },
        searchResultStatus(){
            return this.control.show && this.entity.search.result && this.entity.search.result.length > 0;
        }
    },
    filters: {
        pickIcon(item){
            return `/static/assets/images/entity/png/${item.icon}.png`;
        }
    },
    methods: {
        onToggleBar(){
            this.control.show = !this.control.show; 
        },
        onEntityTreeSelected(data){
            this.entity.search.term = data.alias;
            this.onEntitySearch();
        },
        onEntityClear(){
            this.entity.search.selected = null;
            this.entity.search.result = null;
        },
        onEntitySearch() {

            this.entity.search.loading = true;
            
            if(_.isEmpty(this.entity.search.term)){
                return false;
            }

            let param = encodeURIComponent(this.entity.search.term);
            this.m3.callFS("/matrix/m3graph/entity-search-by-term.js",param).then( rtn=>{
                
                let entitys = rtn.message;

                if(_.isEmpty(entitys)){
                    this.$message({
                        type: "info",
                        message: "没有匹配数据！"
                    })

                    this.entity.search.result = [];

                    this.entity.search.loading = false;

                    return false;
                }

                this.entity.search.result = entitys;

                this.entity.search.result = _.map(this.entity.search.result,(v)=>{
                    return _.extend(v,{ cell: {edge:false} } );
                })

                this.entity.search.loading = false;

            } )
            
        },
        onEntitySelect(){

        },
        onEntityDragStart(item,event){
            event.dataTransfer.setData("Text",JSON.stringify(item));
        },
        onEntityDragEnd(){

        },
        ononEntityDiagnosis(){

        },
        onFileNew(){
            /* this.deleteCells(true);
            this.file.doc = null; */
        },
        onFileOpen(auto){
            /* try{
                
                let editor = inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor;

                // 选择文件打开
                if(!auto){
                    this.file.doc = this.$refs.dfsOpen.node;
                }
                
                let xml = fsHandler.fsContent(this.file.doc.parent, this.file.doc.name);
                let doc = mxUtils.parseXml(xml);
                let codec = new mxCodec(doc);
                codec.decode(doc.documentElement, editor.graph.getModel());
            } catch(err){
                console.error(err);
            } finally {
                this.file.dialogOpen.visible = false;
                inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.toCenter();
            } */
            
        },
        onFileOpenTo(auto){
            /* // 新建图
            let graph = new mxGraph();
            let parent = graph.getDefaultParent();
            
            try{
                
                

                // 选择文件打开
                if(!auto){
                    this.file.doc = this.$refs.dfsOpen.node;
                }
                
                let xml = fsHandler.fsContent(this.file.doc.parent, this.file.doc.name);
                let doc = mxUtils.parseXml(xml);
                let codec = new mxCodec(doc);
                codec.decode(doc.documentElement, graph.getModel());

                // 合并到当前图
                let editor = inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor;
                editor.graph.getModel().mergeChildren(graph.getModel().getRoot(), editor.graph.getDefaultParent());

                // Executes the layout handler
                _.delay(()=>{
                    inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.executeLayout();
                },500)
            } catch(err){
                console.error(err);
            } finally {
                this.file.dialogOpenTo.visible = false;
            } */
            
        },
        onFileSave(){
            /* if(this.file.doc){
                let editor = inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor;
                let enc = new mxCodec(mxUtils.createXmlDocument());
                let node = enc.encode(editor.graph.getModel());
                let xml = mxUtils.getPrettyXml(node);
                let attr = _.extend(this.file.doc, {});
                let rtn = fsHandler.fsNew('file',this.file.doc.parent, this.file.doc.name, xml, attr);
                if(rtn == 1){
                    this.$message({
                        type:"success",
                        message: "保存成功！"
                    })
                }
            } else {
                this.file.dialogSaveAs.visible = true;
            } */
        },
        onFileSaveAs(){
            
            /* let editor = inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor;

            let enc = new mxCodec(mxUtils.createXmlDocument());
            let node = enc.encode(editor.graph.getModel());
            let xml = mxUtils.getPrettyXml(node);
            let attr = {};
            
            let parent = this.$refs.dfsSaveas.node.fullname;
            let name = this.$refs.dfsSaveas.fileName;
            let rtn = fsHandler.fsNew('file',parent, name, xml, attr);
            if(rtn == 1){
                this.$message({
                    type:"success",
                    message: "保存成功！"
                })
                
                this.file.doc = {parent:parent, name:name, fullname: [parent,name].join("/")};
            }

            this.file.dialogSaveAs.visible = false; */
        },
        onSaveAsPdf(){
            // mxUtils.printScreen(inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor.graph);
        },
        onFileClose(){
            // this.deleteCells(true);
            // this.file.doc = null;
        },
        onFilePrint(){
            // inst.app.$refs.graphViewRef.$refs.graphViewContainerInst.model.editor.execute('print');
        },
        onFileDelete(){
            /* if(this.file.doc){

                this.$confirm(`确认要删除该设计文档：${this.file.doc.name}？`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let rtn = fsHandler.fsDelete(this.file.doc.parent,this.file.doc.name);

                    if (rtn == 1){
                        this.$message({
                            type: "success",
                            message: "删除成功！"
                        });
                        this.deleteCells(true);
                        this.file.doc = null;
                        localStorage.setItem("CLASS-DESIGN-OPEN-FILE",'');
                        this.summaryInfo();
                    } else {
                        this.$message({
                            type: "error",
                            message: "删除失败 " + rtn.message
                        })
                    } 
                }).catch(() => {
                    
                });
            } */
        }
    }
}
</script>

<style scoped>
    .path-bar.el-container{
        position: absolute;
        left: 10px;
        top: 10px;
        z-index: 1000;
        background: #f2f2f2;
    }
    .path-bar .el-header{
        text-align: center;
        border: 1px solid #f2f2f2;
    }
    .path-bar .el-button+.el-button {
        margin-left: 0px;
    }
    .path-bar .el-button{
        color: #777777;
        width: 50px;
        border-radius: 0px!important;
    }

    .path-bar .input{
        width: auto;
        min-width: 10vw;
    }

    .path-bar .search span{
        color: #ffffff;
    }

    .path-bar .path-main{
        width:100%;
        height:60vh;
        padding:0px;
        border-top:1px solid #409EFF;
    }
</style>
<style>
    .path-bar .el-radio-button__orig-radio:checked+.el-radio-button__inner {
        color: #252d47;
        background-color: transparent;
        border-color: transparent;
        box-shadow: -1px 0 0 0 #252d47;
    }

    .path-bar .el-radio-button__inner {
        background: transparent;
        border: unset;
    }

    .path-bar .el-radio-button:first-child .el-radio-button__inner {
        border-left: unset;
    }

    .path-bar .el-textarea__inner,
    .path-bar .el-radio-button__inner{
        border:unset;
    }
    .path-bar .el-radio-button__orig-radio:checked+.el-radio-button__inner {
        color: #333333;
        background-color: transparent;
        border-color: #ffffff;
        -webkit-box-shadow: unset;
        box-shadow: unset;
    }
    .path-bar .el-radio-button__orig-radio:checked+.el-radio-button__inner:before {
        content: "";
        position: absolute;
        bottom: -2px;
        left: 29px;
        border-width: 0 6px 6px;
        border-style: solid;
        border-color: #2790e2 transparent;
        display: block;
        width: 0;
        -webkit-transition-duration: 0.3s;
        transition-duration: 0.3s;
        -webkit-transition-property: transform;
        transition-property: transform;
    }
    .path-bar .el-radio-button__orig-radio:checked+.el-radio-button__inner:after {
        content: "";
        position: absolute;
        bottom: -2px;
        left: 30px;
        border-width: 0 5px 5px;
        border-style: solid;
        border-color: #f2f2f2 transparent;
        display: block;
        width: 0;
        -webkit-transition-duration: 0.3s;
        transition-duration: 0.3s;
        -webkit-transition-property: transform;
        transition-property: transform;
    }

    .path-bar .el-radio-group {
        display: flex;
    }

    .path-bar .el-radio-group > .el-radio {
        margin-right: 20px;
    }
</style>