<template>
    <span>
        <span
            v-if="datePanelLabel"
            v-show="datePanelLabel.labels[0].type === 'year' || currentView === 'date' || currentView === 'week'"
            :class="[datePrefixCls + '-header-label']"
            @click="datePanelLabel.labels[0].handler">{{ datePanelLabel.labels[0].label }}</span>
        <template v-if="datePanelLabel && (currentView === 'date'  || currentView === 'week')">{{ datePanelLabel.separator }}</template>
        <span
            v-if="datePanelLabel"
            v-show="datePanelLabel.labels[1].type === 'year' || currentView === 'date' || currentView === 'week'"
            :class="[datePrefixCls + '-header-label']"
            @click="datePanelLabel.labels[1].handler">{{ datePanelLabel.labels[1].label }}</span>
    </span>
</template>

<script>
export default {
    props: {
        datePanelLabel: Object,
        currentView: String,
        datePrefixCls: String
    }
};
</script>
