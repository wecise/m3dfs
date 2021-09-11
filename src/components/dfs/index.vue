<!--
    tree -> view

    view -> tree

-->
<template>
    <el-container>
        <el-main>
            <Split direction="horizontal" :gutterSize="5">
                <SplitArea :size="20" :minSize="0" style="overflow:hidden;background:#f2f2f2;">
                    <DfsTree :root="root" 
                        @node-click="onNodeClick"
                        @dfs-newdir="onNewDir"
                        @dfs-newfile="onNewFile"
                        @dfs-delete="onDfsDelete"></DfsTree>
                </SplitArea>
                <SplitArea :size="80" :minSize="0" style="overflow:hidden;">
                    <ToolBar :root="root" 
                             :rootTitle="rootTitle" 
                             @toggle-view="onToggleView"
                             @dfs-refresh="onRefresh"
                             @select-all="onSelectAll"
                             @node-click="onNodeClick"
                             @dfs-newdir="onNewDir"
                             @dfs-newfile="onNewFile"
                             @dfs-delete="onDfsDelete"
                             @order-by="onOrderBy"
                             @dfs-export="onDfsExport"
                             @dfs-import="onDfsImport"></ToolBar>
                        <component v-bind:is="currentView" 
                            :model="model"
                            :selected="selectedItem"
                            v-if="model"
                            @dblclick.native="onResetStatus"
                            @dbl-click="onLoad"></component>
                </SplitArea>
            </Split>
        </el-main>
    </el-container>
</template>

<script>
    import _ from 'lodash';
    import DfsTree from './dfsTree.vue';
    import ToolBar from './toolBar.vue';
    import gridView from './gridView.vue';
    import thumbnailsView from './thumbnailsView.vue';
    import tableView from './tableView.vue';

    export default {
        data(){
            return {
                root: {parent:"/", fullname: "/"},
                rootTitle: "我的文件",
                currentView: "gridView",
                selectedItem: [],
                model: null
            }
        },
        components:{
            DfsTree,
            ToolBar,
            gridView,
            thumbnailsView,
            tableView
        },
        mounted(){
            this.$nextTick(()=>{
                this.onLoad(null);
            })
        },
        methods:{
            onToggleView(view){
                this.currentView = view;
            },
            onNodeClick(node){
                this.onLoad(node);
            },
            onSelectAll(){
                if(_.isEmpty(this.selectedItem)){
                    this.selectedItem = _.map(this.model,'id');
                } else {
                    this.selectedItem = [];
                }
            },
            onLoad(node){
                
                let fullname = node ? node.fullname : "/";
                this.root = node ? node : {fullname: "/"};
                this.m3.dfsList({fullname:fullname}).then( res=>{
                    this.model = _.orderBy(_.map(res.message,(v,k)=>{

                                    return _.merge(v, {editable: false});

                                }),[v => v.name.toLowerCase(),"ctime"],["asc","desc"]);
                });
            },
            onResetStatus(){

            },
            getMenuByType(item){
                // 根据文件类型返回右键菜单
                let model = {
                    "js": [
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑/运行", icon: "fas fa-play", fun: "editIt" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "html": [
                        { name: "发布应用", icon: "fab fa-codepen", fun: "onNewAppDeploy" },
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "运行", icon: "fas fa-play", fun: "runHtml" },
                        { name: "运行链接", icon: "fas fa-map-marker-alt", fun: "copyUrl" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "htm": [
                        { name: "发布应用", icon: "fab fa-codepen", fun: "onNewAppDeploy" },
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "运行", icon: "fas fa-play", fun: "runHtml" },
                        { name: "运行链接", icon: "fas fa-map-marker-alt", fun: "copyUrl" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info" }
                    ],
                    "md":[
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "打开", icon: "fas fa-play", fun: "openIt", type: null },
                        { name: "打开链接", icon: "fas fa-map-marker-alt", fun: "copyUrl" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "json":[
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "打开", icon: "fas fa-play", fun: "openIt", type: true },
                        { name: "打开链接", icon: "fas fa-map-marker-alt", fun: "copyUrl" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "imap":[
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "ishow":[
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "iflow":[
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "编辑", icon: "fas fa-edit", fun: "editIt" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "dir": [
                        { name: "打开", icon: "fas fa-folder-open", fun: "openIt" },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ],
                    "normal": [
                        { name: "打开", icon: "fas fa-folder-open", fun: "openIt" },
                        { name: "下载", icon: "fas fa-download", fun: "downloadIt", type: true },
                        { name: "删除", icon: "fas fa-trash", fun: "deleteIt", type: true },
                        { name: "共享", icon: "fas fa-share-square", fun: "shareIt" },
                        { name: "属性", icon: "fas fa-info", fun: "info", divided:true }
                    ]
                };

                var menu = null;
                try{
                    if(model[item.ftype]){
                        menu = model[item.ftype];
                    } else {
                        menu = model["normal"];
                    }
                } catch(err){
                    menu = model["normal"];
                } finally{
                    
                    if(_.startsWith(item.parent,'/web')){
                        menu.push({ name: "同步", icon: "fas fa-download", fun: "onSyncToLocal", divided:true});
                    }

                    if(_.endsWith(window.location.pathname,'api') && _.includes(['js'],item.ftype)) {
                        menu.push({ name: "任务", icon: "fas fa-tasks", fun: "", divided:true});
                    }
                }
                return menu;

            },
            onRefresh(node){
                this.onLoad(node);
            },
            onNewDir(node){
                this.$prompt('请输入目录名称', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消'
                }).then(({ value }) => {

                    if(_.isEmpty(value)){
                        this.$message({
                            type: 'warning',
                            message: '请输入目录名称'
                        });
                        return false;
                    }

                    let param = {
                                    parent: node.fullname, 
                                    name: value,
                                    data:{content:null,ftype:'dir',attr:""}
                    };
                    
                    this.m3.dfsNew(param).then(res=>{
                        this.$message({
                            type: "success",
                            message: '新建目录成功'
                        })
                    }).catch(err=>{
                        this.$message({
                            type: "error",
                            message: '新建目录失败：' + err
                        })
                    });

                    
                }).catch(() => {
                    this.$message({
                    type: 'info',
                    message: '取消新建目录操作'
                    });       
                });
            },
            onNewFile(node){
                this.$prompt('请输入文件名称', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消'
                }).then(({ value }) => {
                    
                    if(_.isEmpty(value)){
                        this.$message({
                            type: 'warning',
                            message: '请输入文件名称'
                        });
                        return false;
                    }

                    let param = {
                                    parent: node.fullname, 
                                    name: [value,'md'].join("."),
                                    data:{content:null,ftype:'md',attr:""}
                    };
                    
                    this.m3.dfsNew(param).then(res=>{
                        this.$message({
                            type: "success",
                            message: '新建文件成功'
                        })
                    }).catch(err=>{
                        this.$message({
                            type: "error",
                            message: '新建文件失败：' + err
                        })
                    });
                    
                }).catch(() => {
                    this.$message({
                    type: 'info',
                    message: '取消新建文件操作'
                    });       
                });
            },
            onDfsDelete(node){
                
                this.$confirm(`确定要删除：${node.name}？`, '删除提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    
                    this.m3.dfsDelete({parent:node.parent, name:node.name}).then(()=>{
                        this.$message({
                            type: "success",
                            message: '删除成功'
                        })
                    }).catch(err=>{
                        this.$message({
                            type: "error",
                            message: '删除失败：' + err
                        })
                    });

                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '取消删除操作'
                    });   
                });
            },
            onOrderBy(data){
                let command = JSON.parse(data);
                this.model = _.orderBy(this.model,[command.prop],[command.type]);
            },
            onDfsExport(data){
                this.$confirm(`确认要导出 ${data.parent} 目录下的所有文件?`, '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                }).then(() => {
                    this.$message({
                        type: "info",
                        message: "导出操作将提交至后台，请稍后。。。"
                    })
                }).catch(() => {
                        
                }); 
            },
            onDfsImport(data){
                let file = data.target.files[0];
                    
                const h = this.$createElement;
                this.$msgbox({
                        title: `导入文件确认`, 
                        message: h('span', null, [
                            h('h4', null, `提示：导入后将覆盖原有文件，请确认！`),
                            h('p', null, `文件名称：${file.name}`),
                            h('p', null, `修改时间：${file.lastModifiedDate}`),
                            h('p', null, `文件大小：${file.size}`)
                        ]),
                        showCancelButton: true,
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning',
                        customClass: "fs-custom-msgbox"
                }).then(() => {

                    this.$message({
                        type: "info",
                        message: "导入操作将提交至后台，请稍后。。。"
                    })

                }).catch(() => {
                    this.$message({
                        type: "info",
                        message: "导入操作已取消"
                    })  
                }); 
            }
        }
    }
</script>

<style scoped>
    .el-container{
        height: calc(100vh - 80px);
    }
    .el-main{
        padding:0px;
        overflow: hidden;
    }

    .el-main .content{
        padding: 0px 10px;
        height: calc(100% - 70px);
        margin-top:30px;
        overflow: auto;
        display: flex;
        flex-wrap: wrap;
        justify-content: flex-start;
        align-items: flex-start;
        align-content: flex-start;
        background: #ffffff;
    }
</style>