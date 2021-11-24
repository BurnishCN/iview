<template>
    <div :class="classes">
        <div :class="[prefixCls + '-header']">
            <span v-for="day in headerDays" :key="day">
                {{day}}
            </span>
        </div>
        <ul>
            <li
                v-for="weekDate in cells" :key="weekDate.week"
                @click="handleClick(weekDate, $event)"
                @mouseenter="handleMouseMove(weekDate)"
                :class="getRowCls(weekDate)"
            >
                <span
                    :class="getCellCls(cell)"
                    v-for="(cell, i) in weekDate.dates"
                    :key="String(cell.date) + i"
                >
                    <em>{{ cell.desc }}</em>
                </span>
            </li>
        </ul>

    </div>
</template>
<script>
    import moment from 'moment';
    import { clearHours, isInRange } from '../util';
    import dateUtil from '../../../utils/date'
    import Locale from '../../../mixins/locale';
    import jsCalendar from 'js-calendar';

    import mixin from './mixin';
    import prefixCls from './prefixCls';


    export default {
        mixins: [ Locale, mixin ],

        data () {
            return {
                prefixCls: prefixCls,
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    `${prefixCls}-week`,
                    `${prefixCls}-show-week-numbers`
                ];
            },
            calendar(){
                const weekStartDay = Number(this.t('i.datepicker.weekStartDay'));
                return new jsCalendar.Generator({onlyDays: false, weekStart: weekStartDay});
            },
            headerDays () {
                const weekStartDay = Number(this.t('i.datepicker.weekStartDay'));
                const translatedDays = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'].map(item => {
                    return this.t('i.datepicker.weeks.' + item);
                });
                const weekDays = translatedDays.splice(weekStartDay, 7 - weekStartDay).concat(translatedDays.splice(0, weekStartDay));
                return [''].concat(weekDays)
            },
            cells () {
                const tableYear = this.tableDate.getFullYear();
                const tableMonth = this.tableDate.getMonth();
                const today = clearHours(new Date());    // timestamp of today
                const selectedDays = this.dates.filter(Boolean).map(clearHours);    // timestamp of selected days
                const [minDay, maxDay] = this.dates.map(clearHours);
                const rangeStart = this.rangeState.from && clearHours(this.rangeState.from);
                const rangeEnd = this.rangeState.to && clearHours(this.rangeState.to);

                const isRange = this.selectionMode === 'range';
                const disabledTestFn = typeof this.disabledDate === 'function' && this.disabledDate;

                const weekDateCells = []
                const weekDateSize = 8;

                const cells = this.calendar(tableYear, tableMonth, (cell) => {
                    // normalize date offset from the dates provided by jsCalendar
                    // Comment out this code to fix daylight saving time bug
                    // https://www.cnblogs.com/hamsterPP/p/5415472.html
                    if (cell.date instanceof Date) cell.date.setTime(cell.date.getTime() + cell.date.getTimezoneOffset() * 60000 + 480 * 60 * 1000);
                    //if (cell.date instanceof Date) cell.date.setTime(clearHours(cell.date));

                    const time = cell.date && clearHours(cell.date);
                    const dateIsInCurrentMonth = cell.date && tableMonth === cell.date.getMonth();
                    return {
                        ...cell,
                        type: time === today ? 'today' : cell.type,
                        selected: dateIsInCurrentMonth && selectedDays.includes(time),
                        disabled: (cell.date && disabledTestFn) && disabledTestFn(new Date(time)),
                        range: dateIsInCurrentMonth && isRange && isInRange(time, rangeStart, rangeEnd),
                        start: dateIsInCurrentMonth && isRange && time === minDay,
                        end: dateIsInCurrentMonth && isRange && time === maxDay
                    };
                }).cells.slice(weekDateSize)
                const len = cells.length/weekDateSize

                for(let i = 0; i < len; i++ ) {
                    const weekDates = cells.slice(i * weekDateSize, i * weekDateSize + weekDateSize)
                    const weekDate = {
                        ...weekDates[0],
                        date: weekDates[1].date,
                        type: 'week',
                        year: tableYear,
                        dates: weekDates
                    }

                    weekDateCells.push(weekDate)
                }

                return weekDateCells
            }
        },
        methods: {
            getCellCls (cell) {
                return [
                    `${prefixCls}-cell`,
                    {
                        [`${prefixCls}-cell-today`]: cell.type === 'today',
                        [`${prefixCls}-cell-prev-month`]: cell.type === 'prevMonth',
                        [`${prefixCls}-cell-next-month`]: cell.type === 'nextMonth',
                        [`${prefixCls}-cell-week-label`]: cell.type === 'weekLabel',
                        [`${prefixCls}-cell-range`]: cell.range && !cell.start && !cell.end,
                    }
                ];
            },
            getRowCls (cell) {
                const isActiveWeek = moment(this.value[0]).format('yyyy-W') == `${ cell.year }-${ cell.week }`
                return [
                    `${prefixCls}-row`,
                    {
                        [`${prefixCls}-cell-disabled`]: cell.disabled,
                        [`${prefixCls}-week-row-selected`]: isActiveWeek
                    }
                ];
            },
        }
    };
</script>
