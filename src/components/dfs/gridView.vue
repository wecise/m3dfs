<template>
    <el-checkbox-group v-model="dt.selected" class="dfs-container">
        <el-button type="default" 
                class="dfs-node"
                @dblclick.native="onDblClick(item)"
                @click="onTriggerClick(item)"
                v-for="(item,index) in dt.rows"
                :key="index">
                
                <el-image style="width:34px;height:34px;margin:5px;" :src="item | pickIcon"></el-image>
                <p style="white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;" v-show="item.editable">
                    <el-input v-model="item.name" 
                        type="textarea" 
                        autosize 
                        @keyup.enter.native="$event.target.blur"
                        @blur.self="onRename(item)" :ref="'input_'+item.id"></el-input>
                </p>
                <p style="white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;" @click.self="onSetFocus(item)" v-show="!item.editable">
                    {{item.name}}
                </p>
                <el-checkbox :label="item.id" :ref="'checkBox_'+item.id"></el-checkbox>
        </el-button>
    </el-checkbox-group>
</template>

<script>
    import _ from 'lodash';
    export default {
        props: {
            model: Array,
            selected: Array
        },
        data(){
            return {
                dt:{
                    rows: [],
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
        max-width: 12em;
        width: 12em;
        height:110px;
        border-radius: 10px!important;
        border: unset;
        box-shadow: 0 0px 5px 0 rgba(0, 0, 0, 0.05);
        background: rgb(239, 244, 246);
    }
    .dfs-node + .dfs-node{
        margin: 5px;
    }
</style>

<style>
    .dfs-container .el-checkbox__input{
        float:right;
    }
    .dfs-container .el-checkbox__label{
        display: none;
    }
    .dfs-container .el-checkbox__input > .el-checkbox__inner{
        border:unset;
    }
</style>