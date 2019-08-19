<template>
    <table cellspacing="0" cellpadding="0" border="0" :style="styleObject">
        <colgroup>
            <col v-for="(column, index) in columns" :width="setCellWidth(column)">
        </colgroup>
        <tbody :class="[prefixCls + '-tbody']" ref="tableBody">
            <!-- <template v-for="(row, index) in data"> -->
            <div :style="{height: `${topPlaceholderHeight}px`}"></div>
            <template v-for="(row, index) in optimizeData" >
                <!-- 优化data v-if="optimizeData"-->
                <!-- optimizeData  阻止vue的set和get -->
                <table-tr :draggable="draggable" :row="Object.freeze(row)" :key="rowKey ? row._rowKey : row._index" :prefix-cls="prefixCls"
                    @mouseenter.native.stop="handleMouseIn(row._index)"
                    @mouseleave.native.stop="handleMouseOut(row._index)" @click.native="clickCurrentRow(row._index)"
                    @dblclick.native.stop="dblclickCurrentRow(row._index)">
                    <td v-for="column in columns" :class="alignCls(column, row)" ref="tableTd">
                        <table-cell :fixed="fixed" :prefix-cls="prefixCls" :row="Object.freeze(row)" :key="column._columnKey"
                            :column="column" :natural-index="row._index" :index="row._index"
                            :checked="rowChecked(row._index)" :disabled="rowDisabled(row._index)"
                            :expanded="rowExpanded(row._index)"></table-cell>
                    </td>
                </table-tr>
                <tr v-if="rowExpanded(row._index)" :class="{[prefixCls + '-expanded-hidden']: fixed}">
                    <td :colspan="columns.length" :class="prefixCls + '-expanded-cell'">
                        <Expand :key="rowKey ? row._rowKey : row._index" :row="Object.freeze(row)" :render="expandRender"
                            :index="row._index"></Expand>
                    </td>
                </tr>
            </template>
            <div :style="{height: `${bottomPlaceholderHeight}px`}"></div>
        </tbody>
    </table>
</template>
<script>
    // todo :key="row"
    import TableTr from './table-tr.vue';
    import TableCell from './cell.vue';
    import Expand from './expand.js';
    import Mixin from './mixin';
    import { deepCopy } from '../utils/assist';
    export default {
        name: 'TableBody',
        mixins: [Mixin],
        components: { TableCell, Expand, TableTr },
        props: {
            prefixCls: String,
            styleObject: Object,
            columns: Array,
            data: Array,    // rebuildData
            objData: Object,
            columnsWidth: Object,
            fixed: {
                type: [Boolean, String],
                default: false
            },
            draggable: {
                type: Boolean,
                default: false
            },
            rowKey: {
                type: Boolean,
                default: false
            },
            tableDataIndex: {
                type: [Number, String],
            },
            rowNumber: {
                type: [Number, String],
            },
            rowHeight: {
                type: [Number, String]
            },
            moduleNumber: {
                type: [Number, String]
            },
            scrollTop:{
                type:[Number,String],
                dafault:0,
            }
        },
        data() {
            return {
                optimizeData: null, //rebuild data
                cloneData: null,// clone data
                tableData1: [],
                tableData2: [],
                tableData3: [],
                renderTable1:[],
                renderTable2:[],
                renderTable3:[],
                times0: 0, // 当前是第几轮
                times1: 0,
                times2: -1,
                topPlaceholderHeight: 0,
                // bottomPlaceholderHeight: 0,
                dataHeight: 0,
                totalRowHeight: 0,// 全量渲染有多高
            }
        },
        computed: {
            expandRender() {
                let render = function () {
                    return '';
                };
                for (let i = 0; i < this.columns.length; i++) {
                    const column = this.columns[i];
                    if (column.type && column.type === 'expand') {
                        if (column.render) render = column.render;
                    }
                }
                return render;
            },
            bottomPlaceholderHeight() {
                // console.log(this.totalRowHeight,"bottomPlaceholderHeight",this.topPlaceholderHeight,"topPlaceholderHeight");
                return (this.placeholderHeight - this.topPlaceholderHeight) < 0 ? 0 : this.placeholderHeight - this.topPlaceholderHeight;
            },
            placeholderHeight() {
                //  当前三块总高度
                // if(this.optimizeData && this.optimizeData.length>0){
                    // this.dataHeight =this.moduleNumber * this.rowHeight*3;
                // }
                // console.log(this.totalRowHeight,this.dataHeight)
                return this.totalRowHeight - this.moduleNumber*3 * this.rowHeight; // 占位容器的总高度(上 + 下)
            }
        },
        watch: {
            tableDataIndex: {
                handler(val) {
                    // console.log(val, "val");
                    // let _array = [];
                    // let a = this.tableDataIndex * this.rowNumber > this.data.length ? this.data.length : this.tableDataIndex * this.rowNumber;
                    // if (this.data && this.data.length > 0) {
                    //     for (let i = 0; i < a; i++) {
                    //         _array.push(this.data[i]);
                    //     }
                    //     this.optimizeData = _array;
                    // }
                    // console.log(this.optimizeData, '--', this.data)
                    // this.setTopPlace();
                    if(this.data && this.data.length>0){
                        console.log(val,"index变化了呢",new Date().getTime())
                        this.getTimes();
                        this.setTopPlace();
                        // setTimeout(()=>{
                            this.getTableData();
                            let _array= this.getTable();
                            this.optimizeData = _array
                        // },16)
                             console.log(val,"index渲染结束了呢",new Date().getTime())
                            //滚动卡顿
                           
                        
                    }
                },
                // deep: true,
                immediate: true,
            },
            data: {
                handler(val) {
                    if (val && val.length > 0) {
                        this.totalRowHeight = val.length * this.rowHeight;
                        // console.log(this.totalRowHeight,"totalRowHeight");
                        //insert data.length td
                        // this.optimizeData = this._insert(val.length);
                        // let a = this.tableDataIndex * this.rowNumber > this.data.length ? this.data.length : this.tableDataIndex * this.rowNumber;
                        // let _array = [];
                        // if (this.data && this.data.length > 0) {
                        //     for (let i = 0; i < a; i++) {
                        //         _array.push(this.data[i]);
                        //     }
                        //     this.optimizeData = _array;
                        // }
                        
                        console.log(val,"data变化了")
                        this.getTimes();
                        this.setTopPlace();
                        // setTimeout(()=>{
                            this.getTableData();
                            let _array= this.getTable();
                            this.optimizeData = _array
                        // },16)
                        
                        // console.log(this.getTable(),val,"optimizeData")
                    }
                },
                deep: true,
                immediate: true
            },
            // optimizeData: {
            //     handler(val) {
            //         // console.log(val,"optimizeData")
            //         // if(val && val.length>0){
            //             console.log(val)
            //         //     this.optimizeData = val.slice(0,this.tableDataIndex*this.itemNum);
            //         // }
            //     },
            //     deep: true,
            //     immediate: true,
            // }
        },
        methods: {
            _insert(number) {
                let _array = [];
                for (let i = 0; i < number; i++) {
                    _array.push({});
                }
                this.cloneData = deepCopy(_array);
                // return _array;
            },
            getTimes(){
                let scrollTop = this.scrollTop;
                // console.log(scrollTop,"scrollTop")
                let t0 = 0;
                let t1 = 0;
                let t2 = 0;
                let moduleHeight = this.moduleNumber * this.rowHeight;
                if (scrollTop > moduleHeight) {
                    switch (this.tableDataIndex) {
                        case 0: t0 = parseInt(scrollTop / (moduleHeight * 3)); t1 = t2 = t0; break;
                        case 1: t1 = parseInt((scrollTop - moduleHeight) / (moduleHeight * 3)); t0 = t1 + 1; t2 = t1; break;
                        case 2: t2 = parseInt((scrollTop - moduleHeight * 2) / (moduleHeight * 3)); t0 = t1 = t2 + 1;
                    }
                }
                this.times0 = t0;
                this.times1 = t1;
                this.times2 = t2;
            },
            setTopPlace() {
                let scrollTop = this.scrollTop;
                let moduleHeight = this.moduleNumber * this.rowHeight;
                this.topPlaceholderHeight = parseInt(scrollTop / moduleHeight) * moduleHeight;
                // console.log(this.topPlaceholderHeight,"topPlaceholderHeight")
                
            },
            getTable() {
                //get  table the module number ,group has 3 module
                let _index = this.tableDataIndex;
                let _number = this.rowNumber;
                
                if (_index === 0) return Object.preventExtensions([...this.tableData1,...this.tableData2,...this.tableData3]);
                if (_index === 1) return Object.preventExtensions([...this.tableData2,...this.tableData3,...this.tableData1]);
                else return Object.preventExtensions([...this.tableData3,...this.tableData1,...this.tableData2]);
            },
            getTableData() {
                // get table  data
                // let _index = this.tableDataIndex;
                // let t0, t1, t2;
                // let moduleHeight = this._rowHeight * this.moduleNumber;
                // switch (_index) {
                //     case 0: t0 = parseInt(this.$parent.scrollTop / (moduleHeight * 3)); t1 = t2 = t0; break;
                //     case 1: t1 = parseInt((this.$parent.scrollTop - moduleHeight) / (moduleHeight * 3)); t0 = t1 + 1; t2 = t1; break;
                //     case 2: t2 = parseInt((this.$parent.scrollTop - moduleHeight * 2) / (moduleHeight * 3)); t0 = t1 = t2 + 1; break;
                // }
                // this.times0 = t0;
                // this.times1 = t1;
                // this.times2 = t2;
                let data =Object.freeze(deepCopy(this.data));
                let count1 = this.times0 * this.moduleNumber * 3;
                let count2 = this.times1 * this.moduleNumber * 3;
                let count3 = this.times2 * this.moduleNumber * 3;
                this.tableData1 =Object.freeze(data.slice(count1, count1 + this.moduleNumber));
                this.tableData2 = Object.freeze(data.slice(count2 + this.moduleNumber, count2 + this.moduleNumber * 2));
                this.tableData3 = Object.freeze(data.slice(count3 + this.moduleNumber * 2, count3 + this.moduleNumber * 3));
                //    console.log(this.tableData3,"tableData3")
            },
            rowChecked(_index) {
                return this.objData[_index] && this.objData[_index]._isChecked;
            },
            rowDisabled(_index) {
                return this.objData[_index] && this.objData[_index]._isDisabled;
            },
            rowExpanded(_index) {
                return this.objData[_index] && this.objData[_index]._isExpanded;
            },
            handleMouseIn(_index) {
                this.$parent.handleMouseIn(_index);
            },
            handleMouseOut(_index) {
                this.$parent.handleMouseOut(_index);
            },
            clickCurrentRow(_index) {
                this.$parent.clickCurrentRow(_index);
            },
            dblclickCurrentRow(_index) {
                this.$parent.dblclickCurrentRow(_index);
            },

        }
    };
</script>