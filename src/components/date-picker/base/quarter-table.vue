<template>
    <div :class="classes">
        <span
            :class="getCellCls(cell)"
            v-for="cell in cells"
            :key="cell.text"
            @click="handleClick(cell)"
            @mouseenter="handleMouseMove(cell)"

        >
            <em>{{ cell.text }}</em>
        </span>
    </div>
</template>
<script>
    import { clearHours } from '../util';
    import { deepCopy } from '../../../utils/assist';
    import Locale from '../../../mixins/locale';
    import mixin from './mixin';
    import prefixCls from './prefixCls';

    export default {
        mixins: [ Locale, mixin ],
        props: {/* in mixin */},
        computed: {
            classes() {
                return [
                    `${prefixCls}`,
                    `${prefixCls}-quarter`
                ];
            },
            cells () {
                let cells = [];
                const cell_tmpl = {
                    text: '',
                    selected: false,
                    disabled: false
                };

                const tableYear = this.tableDate.getFullYear();
                const selectedDays = this.dates.filter(Boolean).map(date => clearHours(new Date(date.getFullYear(), date.getMonth(), 1)));
                const focusedDate = clearHours(new Date(this.focusedDate.getFullYear(), this.focusedDate.getMonth(), 1));

                for (let i = 0; i < 4; i++) {
                    const cell = deepCopy(cell_tmpl);
                    // 每一季度的日期设置为当季的第一天
                    cell.date = new Date(tableYear, i * 3, 1);
                    cell.quarter = i + 1
                    cell.text = `Q${ cell.quarter }`;
                    const day = clearHours(cell.date);
                    cell.disabled = typeof this.disabledDate === 'function' && this.disabledDate(cell.date) && this.selectionMode === 'quarter';
                    cell.selected = selectedDays.includes(day);
                    cell.focused = day === focusedDate;
                    cells.push(cell);
                }

                return cells;
            }
        },
        methods: {
            getCellCls (cell) {
                return [
                    `${prefixCls}-cell`,
                    {
                        [`${prefixCls}-cell-selected`]: cell.selected,
                        [`${prefixCls}-cell-disabled`]: cell.disabled,
                        [`${prefixCls}-cell-focused`]: cell.focused,
                        [`${prefixCls}-cell-range`]: cell.range && !cell.start && !cell.end
                    }
                ];
            },
        }
    };
</script>
