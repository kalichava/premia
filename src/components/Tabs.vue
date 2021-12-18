<template>
    <div class="tabs tabs--chart">
        <div
            v-for="(tab, index) in $props.tabs" :key="index"
            class="tabs__tab"
            :class="{ 'active': $props.active === index }"
            @click="$emit('changeTab', { index, chart: $props.chart })"
        >
            {{ tab.name }}
        </div>
    </div>
</template>

<script>
export default {
    props: {
        tabs: {
            type: Array,
            default: () => []
        },
        chart: {
            type: String,
        },
        active: {
            type: Number,
            default: 0
        }
    },
}
</script>

<style lang="scss">
.tabs {
    display: flex;
    border: 1px solid $colorStroke;
    border-radius: $radiusOuter;
    padding: 4px;
    background: rgba($black, .5);
    &.tabs--chart {
        position: absolute;
        top: 10px;
        right: 10px;
        z-index: 4;
    }
}
.tabs__tab {
    background: transparent;
    border-radius: $radiusInner;
    padding: 6px 18px;
    font-size: 14px;
    color: $colorTextLight;
    cursor: pointer;
    transition: .3s;
    &:hover {
        color: $colorTextLighter;
    }
    &.active {
        background: rgba($blue, .2);
        color: $blue;
        pointer-events: none;
        font-weight: bold;
    }
}
</style>