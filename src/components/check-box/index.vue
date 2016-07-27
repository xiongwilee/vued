<template>
    <span class="check-box">
        <slot name="left"></slot>
        <span @click.stop="click">
            <span class="checkbox {{className}} {{typeClass}} {{disClass}}"></span>
            <span v-if="label">{{label}}</span>
        </span>
        <slot name="right" class="test"></slot>
    </span>
</template>

<script>
export default {
    props: {
        value: {
            twoWay: true,
            type: Boolean,
            default: false
        },
        type: {
            type: String,
            default: 'circle'
        },
        label: {
            type: String,
            default: ''
        },
        disabled: {
            type: Boolean,
            default: false
        },
        disabledClass: {
            type: String
        }
    },
    computed: {
        className () {
            return (this.value ? 'checkbox-checked' : '')
        },
        typeClass () {
            return (this.type ? 'checkbox-' + this.type : '')
        },
        disClass () {
            return this.disabled ? this.disabledClass || 'checkbox-disabled' : ''
        }
    },
    methods: {
        click () {
            if (this.disabled) return
            this.value = !this.value;
        }
    }
}
</script>

<style lang="less">
@import '../../styles/weui/base/fn.less';
.check-box{
    font-size: 15px;
    .checkbox{
        position: relative;
        display: inline-block;
        width: 0.9em;
        height: 0.9em;
    }

    .checkbox-circle{
        border: 1px solid @globalThemeColor;
        background-color: #fff;
        vertical-align: text-bottom;

        &.checkbox-checked:after{
            content: '';
            position: absolute;
            display: inline-block;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            width: .5em;
            height: .5em;
            margin: auto;
            background-color: @globalThemeColor;
        }

        &,&.checkbox-checked:after{
            border-radius: 50%;
        }
    }

    .checkbox-tick{
        font-family: weui;
        font-style: normal;
        font-weight: 400;
        speak: none;
        color: #188bd3;
        &:before{
            content: '\EA01';
        }

        &.checkbox-checked{
            &:before{
                content: '\EA06';
            }
        }
    }

    .checkbox-disabled{
        color: #ccc;
        border-color: #ccc;
        &.checkbox-circle:after{
            background-color: #ccc;
        }
    }
}

</style>