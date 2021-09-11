<template>
    <el-checkbox-group v-model="dt.selected" class="content dfs-thumbnails-container">
        <el-button type="default" 
                class="dfs-node"
                @dblclick.native="onDblClick(item)"
                @click="onTriggerClick(item)"
                v-for="item in dt.rows"
                :key="item.id">
                <div class="dfs-menu">    
                    <el-dropdown trigger="hover" placement="top-start" @command="onMenuCommand">
                            <span class="el-dropdown-link">
                                <i class="el-icon-arrow-down el-icon--right"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item 
                                    :command="{fun:menuItem.fun,param:item,type:menuItem.type?menuItem.type:false}" 
                                    v-for="(menuItem,index) in getMenuByType(item)" 
                                    :key="index"
                                    :divided="menuItem | pickDivided">
                                    {{menuItem.name}}
                                </el-dropdown-item>
                            </el-dropdown-menu>
                    </el-dropdown>
                </div>
                <el-image style="width:64px;margin:5px;" :src="item | pickIcon"></el-image>
                <div class="dfs-props">
                    <p v-show="item.editable">
                        <el-input v-model="item.name" 
                            type="textarea" 
                            autosize
                            @keyup.enter.native="$event.target.blur"
                            @blur.self="onRename(item)" :ref="'input_'+item.id"></el-input>
                    </p>
                    <p @click.self="onSetFocus(item)" v-show="!item.editable">
                        名称： {{item.name}}
                    </p>
                    <p>作者：{{item.author}}</p>
                    <p>创建：{{moment(item.ctime).format("YYYY-MM-DD hh:mm:ss")}}</p>
                    <p>
                        标签
                        <TagView domain='files' :model.sync="item.tags" :id="item.id" limit="1"></TagView>
                    </p>
                    <p>描述：{{item|pickRemark}}</p>
                </div>
                <el-checkbox :label="item.id" :ref="'checkBox_'+item.id"></el-checkbox>
        </el-button>
    </el-checkbox-group>
</template>

<script>
    import _ from 'lodash';
    import TagView from '../tags/TagView.vue';

    export default {
        props: {
            model: Array,
            selected: Array
        },
        components:{
            TagView
        },
        data(){
            return {
                dt:{
                    rows: [],
                    columns: [],
                    selected: []
                }
            }
        },
        watch: {
            model(val,oldVal){
                this.dt.rows = [...this.model];
            },
            selected:{
                handler(val){
                    this.dt.selected = val;
                },
                immediate:true
            }
        },
        filters: {
            pickName(item){

                if (_.isEmpty(item)) return '';

                let _name = _.head(item.name.split("."));

                if(!_.isEmpty(_name)){
                    _name = _.truncate(_name, {
                        'length': 9
                    });
                }

                return _name;
            },
            pickIcon(item){
                // extend || ...
                if( item.fullname === '/extend' ){
                    return `/static/assets/images/files/png/dir-lock.png`;
                } else {

                    try {
                        return _.attempt(JSON.parse.bind(null, item.attr)).icon || `/static/assets/images/files/png/${item.ftype}.png`;
                    }
                    catch(error){
                        return `/static/assets/images/files/png/${item.ftype}.png`;
                    }
                }

            },
            pickTags(item){

                if(item.tags){
                    return item.tags.join(",");
                } else {
                    return "";
                }
            },
            pickRemark(item){
                if (!_.isEmpty(item.attr) || !_.isEqual(item.attr, 'null')) {
                    return _.attempt(JSON.parse.bind(null, item.attr)).remark;
                }
            },
            pickDivided(item){
                try{
                    return item.divided;
                } catch(err) {
                    return false;
                }
            }
        },
        created(){
            this.dt.rows = [...this.model];
        },
        methods: {
            onDblClick(item){
                this.$emit('dbl-click',item);
            },
            onTriggerClick(item){
                this.$refs['checkBox_'+item.id][0].$el.click();
            },
            getMenuByType(item){
                let parent = this.$parent.$parent.$parent.$parent.$parent;
                return parent.getMenuByType(item);
            },
            onMenuCommand(){

            },
            onSetFocus(item){
                setTimeout(()=>{
                    item.editable = true;
                    setTimeout(()=>{
                        this.$refs['input_'+item.id][0].$refs.textarea.focus();
                    },500)
                },500)
            },
            onRename(item){
                if(_.find(this.model,{id:item.id}).name != item.name){
                    this.$root.$refs.viewRef.$children[0].onRename(_.find(this.model,{id:item.id}), item);
                } else {
                    item.editable = false;
                }
            }

        }

    }
</script>

<style scoped>
    .dfs-node{
        max-width: 25em;
        width: 25em;
        height: 230px;
        border-radius: 10px!important;
        margin: 5px;
        border: unset;
        box-shadow: 0 0px 5px 0 rgba(0, 0, 0, 0.05);
        background: rgb(239, 244, 246);
        margin: 10px
    }
    .dfs-props > p{
        white-space:nowrap;
        overflow:hidden;
        text-overflow:ellipsis;
        margin:5px;
        text-align:left;
    }

    .dfs-menu{
        position: relative;
        right: -130px;
    }
</style>

<style>
    .dfs-thumbnails-container .el-checkbox__input>.el-checkbox__inner {
        right: -135px;
        bottom: 5px;
    }
    .dfs-thumbnails-container .el-checkbox__label{
        display: none;
    }
    .dfs-thumbnails-container .el-checkbox__input > .el-checkbox__inner{
        border:unset;
    }
</style>