<template>
    <div class="content dfs-table-container">
        <el-table style="width: 100%"
            :data="dt.rows"
            stripe
            highlight-current-row="true"
            @selection-change="onSelectionChange"
            ref="table">
            <el-table-column type="selection" align="center"></el-table-column> 
            <template v-for="(item,index) in dt.columns">
                <el-table-column
                    sortable 
                    show-overflow-tooltip
                    :key="index"
                    :prop="item.field"
                    :label="item ? item.title : ''"
                    :width="item.width"
                    v-if="item.visible">
                        <template slot-scope="scope">
                            <div v-html='item.render(scope.row, scope.column, scope.row[item.field], scope.$index)' 
                                v-if="typeof item.render === 'function'">
                            </div>
                            <div v-else-if="item.field==='name'">
                                
                                <el-link type="info" :underline="false" @dblclick.native.prevent="forward(scope.row)" v-show="scope.row.editable">
                                    <el-input v-model="scope.row.name" 
                                        type="textarea" 
                                        autosize 
                                        @keyup.enter.native="$event.target.blur"
                                        @blur.self="onRename(scope.row, scope.$index)" :ref="'input_'+scope.row.id"></el-input>
                                </el-link>

                                <el-link type="info" 
                                    :underline="true" 
                                    @dblclick.native.prevent="forward(scope.row)" 
                                    @click.native.prevent="onSetFocus(scope.row)" 
                                    v-show="!scope.row.editable">
                                    <i class="el-icon-folder-opened" style="color: #ff9800;font-size: 16px;" v-if="scope.row.ftype==='dir'"></i> 
                                    <i class="el-icon-tickets" style="color: #2196f3;font-size: 16px;" v-else></i> 
                                    {{scope.row.name}}
                                </el-link>
                                
                            </div>
                            <div v-else-if="item.field==='tags'">
                                <mx-tag domain='files' :model.sync="scope.row.tags" :id="scope.row.id" limit="1"></mx-tag>
                            </div>
                            <div v-else>
                                {{scope.row[item.field]}}
                            </div>
                            
                        </template>
                </el-table-column>
            </template>
        </el-table>
    </div>
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
                    columns: []
                }
            }
        },
        watch: {
            model:{
                handler(val){
                    _.extend(this.dt, { rows: val } );
                },
                deep:true
            },
            selected:{
                handler(val){
                    this.dt.selected = val;
                },
                immediate:true
            }
        },
        created(){     
            this.m3.callFS("/matrix/m3dfs/fs_data.js", null).then(res=>{
                _.extend(this.dt, {columns: _.map(res.message.columns, function(v){
                        
                if(_.isUndefined(v.visible)){
                    _.extend(v, { visible: true });
                }

                if(!v.render){
                    return v;
                } else {
                    return _.extend(v, { render: eval(v.render) });
                }
                
            })});
            
            _.extend(this.dt, { rows: [...this.model] } );
            });
        },
        methods:{
            forward(row){
                this.$emit('dbl-click', row);
            },  
            onCellClick(row,column,cell,event){

            },
            onSelectionChange(val){
                this.$root.$refs.viewRef.$children[0].selectedItem = _.map(val,'id');
            },
            onSetFocus(item){
                _.forEach(this.dt.rows, (v)=>{
                    if(v.id != item.id){
                        v.editable = false;
                    }
                })
                setTimeout(()=>{
                    item.editable = true;
                    setTimeout(()=>{
                        this.$refs['input_'+item.id][0].$refs.textarea.focus();
                    },500)
                },500)
            },
            onRename(item,index){
                if(this.model[index].name == item.name){
                    item.editable = false;
                } else {
                    this.$root.$refs.viewRef.$children[0].onRename(this.model[index], item);
                }
            }
        }

    }
</script>