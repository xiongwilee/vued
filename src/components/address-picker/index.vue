<template>
    <div class="picker-board" v-show="show">
        <div class="weui_mask_transition" :class="{'weui_fade_toggle': show}" :style="{display: showMask ? 'block' : 'none'}"></div>
        <div class="picker-wrapper">
            <slot name="modal"></slot>
            <div class="picker-box" :class="{'picker-toggle': show}">

                <div class="address-title">
                    <span class="title">请选择省市区</span>   
                    <icon type="clear" @click="show = false" class="close"></icon>
                </div>

                <div class="address-tabs ui-border-b">
                    <span class="address-tab" :class="{'active': $index === curTab,'arrow':$index !== 2}" 
                          @click="selectThisTab($index)" v-for="tab in 3">
                        {{area[$index].area_name || area[$index].intro}}
                    </span>
                </div>

                <div class="address-list" v-on:scroll.prevent>
                    <div class="address-item ui-border-b" :class="{'active': +m.area_id === +area[curTab].area_id}"
                         @click="selectThisArea(m)" v-for="m in curList">
                        {{m.area_name}}
                    </div>
                </div>
            </div>
        </div>

        <loading :show="loading" :text="loadingText"></loading>

    </div>
</template>

<script>

import Icon from '../icon/'
import Flexbox from '../flexbox/'
import FlexboxItem from '../flexbox-item/'

import Loading from '../loading/'

export default {
    components: {
        Icon,
        Flexbox,
        FlexboxItem,
        Loading
    },
    ready () {

        // 获取首列地址
        this.getArea(0)
    },
    data () {
        return {

            // 初始化临时地区数据
            area: [
                {parent_id: 0, area_name:'',area_id:'',intro:'1.选择省'},
                {parent_id: '',area_name:'',area_id:'',intro:'2.选择市'},
                {parent_id: '',area_name:'',area_id:'',intro:'3.选择区'}
            ],

            // 初始化当前tab
            curTab: 0,

            // 父级为key的包含数组MAP
            listMap: {},

            // 是否防止滚动
            isHold: false,

            // 加载提示
            loadingText: '加载中...',
            // 地址加载状态
            loading: false,
        }
    },
    props: {
        areaSync: {
            type: Array,
            twoWay: true
        },
        show: {
            type: Boolean,
            default: false
        },
        showMask: {
            type: Boolean,
            default: true
        },
        areaUrl: {
            type: String,
            default: '/home/area_new'
        }
    },
    computed: {
        curList () {
            // 返回：当前tab对应地区所属[省/市]包含的全部地区
            return this.listMap[ this.area[this.curTab].parent_id ] || []
        }
    },
    methods: {
        selectThisArea (area) {
            let index = +this.curTab;

            // 如果是同一个，不理
            if (+this.area[index].area_id === +area.area_id) return

            // 如果是更改前列地址，则先清除后列地址
            if (index < this.area.length - 1) {
                for (var i = this.area.length - 1; i > index; i--) {
                    this.area[i].parent_id = '';
                    this.area[i].area_name = '';
                    this.area[i].area_id = '';
                }
            }

            // 给area[current]赋值
            this.area[index].area_name = area.area_name
            this.area[index].area_id = area.area_id

            // 选择地址
            switch(index){
                case 0: ;
                case 1: {
                    // 给area[next]赋 parent_id 值
                    this.area[index+1].parent_id = area.area_id

                    // 如果下一级数据没有，则请求
                    // 这里的数据判断迁移至对this.curTab的watch，所以不需要再做判断
                    // if (!this.listMap.hasOwnProperty(area.area_id)) {
                    //     this.getArea(area.area_id)
                    // }

                    // tab前进一次
                    this.curTab++
                    break;
                }
                case 2: {
                    // 给areaSync赋值，并关闭地址选择框
                    this.areaSync = JSON.parse(JSON.stringify(this.area));
                    this.show = false;
                    break;
                }
            }
        },
        selectThisTab (index) {
            // 切换tab
            let parItem = this.area[index-1] || {};

            if (this.area[index].area_id || parItem.area_id) {
                this.curTab = index
            }
        },
        getArea (area_id) {
            // 获取首列地址
            this.loading = true;
            this.$http.get(this.areaUrl, {
                parent_id: area_id || 0
            }, ({data})=>{
                this.loading = false;
                this.$set('listMap['+area_id+']', data.slice(0))
            }, ()=>{
                this.loading = false;
            })
        }
    },
    watch: {
        show (nVal, oVal) {
            if (!nVal) { return }

            let maxTab = 0, tempArea = {};
            for (var i = this.area.length - 1; i >= 0; i--) {
                let intro = this.area[i].intro;

                tempArea = this.areaSync[i] || {};

                // 初始化本次显示的地区选择数据
                if (tempArea && tempArea.parent_id !== '' 
                             && tempArea.area_name 
                             && tempArea.area_id ) {
                    // 如果明确是一条真正的地址数据，则初始化记录
                    this.$set('area['+i+']', {
                        area_id: tempArea.area_id,
                        area_name: tempArea.area_name,
                        parent_id: tempArea.parent_id,
                        intro: intro
                    })
                    // 取最大可见tab
                    maxTab = Math.max(maxTab, i);
                } else {
                    this.$set('area['+i+']', {
                        area_id: '',
                        area_name: '',
                        parent_id: (+i ? '' : 0),
                        intro: intro
                    })
                }
            }
            this.curTab = maxTab;
        },
        curTab () {
            let parendtId = this.area[this.curTab].parent_id
            if (!this.listMap.hasOwnProperty(parendtId)){
                this.getArea(parendtId)
            }
        }
    }
}
   
</script>

<style lang="less">
@import '../../styles/weui/base/fn.less';
    .picker-board{
        position: fixed;
        // z-index: 1;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
    }

    .picker-wrapper{
        position: fixed;
        z-index: 1002;
        width: 100%;
        bottom: 0;
        left: 0;
    }

    .picker-box{
        position: relative;
        margin-top: 20px;
        line-height: 2.5;
        font-size: 16px;
        background-color: #fff;

        backface-visibility: hidden;
        transform: translateY(100%);
        transition: -webkit-transform 0.3s;
        transition: transform .3s;
        transition: transform 0.3s, -webkit-transform 0.3s;

        pointer-events: none;
        height: 0;

        &.picker-toggle {
            height: auto;
            transform: translate(0);
            pointer-events: auto;
        }

        .address-title{
            text-align: left;
            padding-left: 10px;
            background-color: #ffffff;
            line-height: 45px;
            .title{
                font-size: 16px;
            }
            .close{
                &:before{
                    font-size: 20px;
                }
                float: right;
                margin-right: 1em;
            }
        }

    }

    .address-tabs{
        font-size: 14px;
        line-height: 3;
        user-select: none;
        background-color: #eeeeee;
        border-top: 1px solid #dadada;
        border-bottom: 1px solid #dadada;
        overflow: hidden;
        .address-tab{
            position: relative;
            outline: none;
            display: inline-block;
            text-align: center;
            width: 33.33%;
            &.active{
                color: @globalThemeColor;
            }
            &.arrow:before{
                content: " ";
                height: 40px;
                width: 40px;
                border: 1px solid #dddddd;
                border-bottom: 0px;
                border-left: 0px;
                display: inline-block;
                transform: rotate(45deg);
                position: absolute;
                top: 0px;
                right: -5px;
            }
        }
    }

    .address-list{
        position: relative;
        width: 100%;
        height: 20em;
        font-size: 14px;
        overflow: hidden;
        overflow-y: auto;
        background-color: #eeeeee;
        .address-item{
            text-align: center;

            &.active{
                color: @globalThemeColor;
            }
        }
    }
    
</style>