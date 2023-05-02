<template>
    <div class="calendar">
        <div class="calendar-header">
            <button class="calendar-btn" @click="previousMonth">&lt;</button>
            <h2 class="calendar-title">{{ currentMonth }}</h2>
            <button class="calendar-btn" @click="nextMonth">&gt;</button>
        </div>
        <div class="calendar-grid">
            <div class="calendar-day">Пн</div>
            <div class="calendar-day">Вт</div>
            <div class="calendar-day">Ср</div>
            <div class="calendar-day">Чт</div>
            <div class="calendar-day">Пт</div>
            <div class="calendar-day">Сб</div>
            <div class="calendar-day">Вс</div>
            <div v-for="(date, index) in calendarDates" :key="index"
                :class="{ 'calendar-date': true, 'today': isToday(date), 'selected': isSelected(date), 'disabled': isDisabled(date) }"
                @click="selectDate(date)">
                <span class="date-number">{{ (date instanceof Date && !isNaN(date.getTime())) ? date.getDate() : ''
                }}</span>
                <div class="notes">
                    <span v-for="(note, index) in getNotesForDate(date)" :key="index" class="note">{{ note.text }}</span>
                </div>
                <form @submit.prevent="addNote">
                    <input type="text" v-model="state.newNote" placeholder="Добавить заметку">
                    <button type="submit" class="plus">+</button>
                </form>
            </div>
        </div>
        <div v-if="state.continueFrom" class="continue-from">Продолжение с {{ state.continueFrom }}</div>
    </div>
</template>

<script lang="ts" setup>
import { reactive } from 'vue';

interface Note {
    id: number;
    date: string;
    text: string;
}

interface State {
    currentDate: Date;
    selectedDate: Date | null;
    daysOfWeek: string[];
    monthsOfYear: string[];
    notes: Note[];
    newNote: string;
    currentMonth: string;
    continueFrom?: string;
}



const state = reactive<State>({
    currentDate: new Date(),
    selectedDate: null,
    daysOfWeek: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
    monthsOfYear: [
        'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
        'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь',
    ],
    notes: [],
    newNote: '',
    currentMonth: '',
    continueFrom: undefined,
});


const updateCurrentMonth = (state: State) => {
    const month = state.monthsOfYear[state.currentDate.getMonth()];
    const year = state.currentDate.getFullYear();
    state.currentMonth = `${month} ${year}`;
};

updateCurrentMonth(state); // initialize currentMonth

const previousMonth = () => {
    state.currentDate.setMonth(state.currentDate.getMonth() - 1);
    updateCurrentMonth(state);
};

const nextMonth = () => {
    state.currentDate.setMonth(state.currentDate.getMonth() + 1);
    updateCurrentMonth(state);
};

const isToday = (date: Date | null): boolean => {
    if (date === null) {
        return false;
    }
    const today = new Date();
    return date.getDate() === today.getDate() && date.getMonth() === today.getMonth() && date.getFullYear() === today.getFullYear();
};

const isSelected = (date: Date | null): boolean => {
    return date !== null && state.selectedDate !== null && date.getTime() === state.selectedDate.getTime();
};

const isDisabled = (date: Date | null): boolean => {
    if (date === null) {
        return false;
    }
    const today = new Date();
    return date.getMonth() !== state.currentDate.getMonth() || (date.getTime() < today.getTime() && date.getMonth() === today.getMonth() && date.getFullYear() === today.getFullYear());
};

const getNotesForDate = (date: Date): Note[] => {
    if (date === null) {
        return [];
    }
    const dateString = date ? date.toISOString().split('T')[0] : '';

    return state.notes.filter((note) => note.date === dateString);
};


const addNote = () => {
    const newNote = state.newNote.trim();
    if (newNote !== '') {
        const date = state.selectedDate as Date;
        const dateString = date ? date.toISOString().split('T')[0] : '';

        const id = state.notes.length > 0 ? state.notes[state.notes.length - 1].id + 1 : 0;
        state.notes.push({ id, date: dateString, text: newNote });
        state.newNote = '';
    }
};

const selectDate = (date: Date | null) => {
    if (date === null || isDisabled(date)) {
        return;
    }
    state.selectedDate = date;
};

const getCalendarDates = (): (Date | null)[] => {
    const year = state.currentDate.getFullYear();
    const month = state.currentDate.getMonth();
    const firstDayOfMonth = new Date(year, month, 1);
    const lastDayOfMonth = new Date(year, month + 1, 0);
    const dates = [];
    // Add dates from previous month to fill up first week
    const firstDayOfWeek = firstDayOfMonth.getDay() === 0 ? 7 : firstDayOfMonth.getDay(); // Sunday is 0, Monday is 1, ..., Saturday is 6
    const daysToAdd = firstDayOfWeek - 1;
    const previousMonth = new Date(year, month, 0);
    for (let i = previousMonth.getDate() - daysToAdd + 1; i <= previousMonth.getDate(); i++) {
        dates.push(null);
    }
    // Add dates from current month
    for (let i = 1; i <= lastDayOfMonth.getDate(); i++) {
        dates.push(new Date(year, month, i));
    }
    // Add dates from next month to fill up last week
    const lastDayOfWeek = lastDayOfMonth.getDay() === 0 ? 7 : lastDayOfMonth.getDay();
    const daysToFill = 7 - lastDayOfWeek;
    const nextMonth = new Date(year, month + 1, 1);
    for (let i = 1; i <= daysToFill; i++) {
        dates.push(null);
    }
    // Save the last date in the calendar so we can continue from it if needed
    const lastDate = dates[dates.length - 1] as Date;
    state.continueFrom = lastDate ? lastDate.toISOString().split('T')[0] : undefined;
    return dates;
};

const calendarDates = getCalendarDates();
</script>
  
<style lang="less" scoped>
@font-family-base: 'Roboto', sans-serif;

.plus[type="submit"] {

    background-color: #c037fb;
    color: #fff;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s ease-in-out;

    &:hover {
        background-color: darken(#be7a95, 10%);
    }


}

.calendar {
    font-family: @font-family-base;
    max-width: 100%;
    margin: 0 auto;
    background-color: #fff;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    overflow: hidden;

    &-header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-bottom: 20px;
        padding: 20px;
        background-color: #50145f;
        color: #fff;

        .mirror {
            transform: scaleX(-1);
        }

        button {
            background-color: transparent;
            border: none;
            cursor: pointer;
            transition: all 0.3s ease-in-out;
            color: #fff;
            font-size: 20px;
            font-weight: bold;
            margin-left: 20px;
            position: relative;
            padding: 10px;

            &::before {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background-color: rgba(255, 255, 255, 0.1);
                z-index: -1;
                opacity: 0;
                transition: opacity 0.3s ease-in-out;
            }

            &:hover::before {
                opacity: 1;
            }
        }
    }

    &-title {
        font-size: 30px;
        font-weight: bold;
        text-transform: uppercase;
        margin: 0;
    }

    &-grid {
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        grid-gap: 20px;
        padding: 30px;
        border-top: 1px solid #eee;
        border-left: 1px solid #eee;

        &-day {
            font-size: 20px;
            font-weight: bold;
            text-align: center;
            text-transform: uppercase;
            color: #333;
        }

        &-date {
            border-radius: 50%;
            width: 60px;
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.3s ease-in-out;
            position: relative;
            font-size: 20px;
            color: #333;
            text-align: center;

            &.today {
                background-color: #ff0066;
                color: #fff;
            }

            &.selected {
                background-color: #ff0066;
                color: #fff;

                &::after {
                    content: '';
                    position: absolute;
                    top: 50%;
                    left: 50%;
                    transform: translate(-50%, -50%);
                    width: 80%;
                    height: 80%;
                    border-radius: 50%;
                    background-color: rgba(255, 255, 255, 0.5);
                    z-index: -1;
                }
            }

            &:hover {
                background-color: #eee;
            }
        }

        &-disabled {
            opacity: 0.5;
            pointer-events: none;
        }
    }

    input[type="text"] {
        border: none;
        background-color: transparent;
        border-bottom:
            1px solid #eee;
        padding: 10px;
        font-size: 20px;
        font-family: @font-family-base;
        width: 100%;
        box-sizing: border-box;
        margin-bottom: 20px;
        color: #333;

        &:focus {
            outline: none;
            border-bottom-color: #ff0066;
        }
    }

    &-actions {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 20px;

        button {
            background-color: #ff0066;
            border: none;
            color: #fff;
            font-size: 20px;
            font-weight: bold;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s ease-in-out;

            &:hover {
                background-color: darken(#ff0066, 10%);
            }
        }
    }

    &-event {
        background-color: #eee;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 20px;

        &-title {
            font-size: 20px;
            font-weight: bold;
            margin: 0;
        }

        &-date {
            font-size: 16px;
            margin-top: 5px;
            margin-bottom: 10px;
        }

        &-description {
            font-size: 16px;
            margin-bottom: 0;
        }
    }

    &-form {
        display: flex;
        flex-direction: column;
    }

    &-form-row {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;

        label {
            font-size: 20px;
            font-weight: bold;
            margin-right: 20px;
        }

        input[type="text"],
        select {
            border: none;
            background-color: transparent;
            border-bottom: 1px solid #eee;
            padding: 10px;
            font-size: 16px;
            font-family: @font-family-base;
            width: 100%;
            box-sizing: border-box;
            color: #333;

            &:focus {
                outline: none;
                border-bottom-color: #ff0066;
            }
        }

        select {
            appearance: none;
            background-image: url('data:image/svg+xml;utf8,<svg viewBox="0 0 24 24" fill="%23333%"><path d="M7.41 8.59L12 13.17l4.59-4.58L18 10l-6 6-6-6 1.41-1.41z"/></svg>');
            background-repeat: no-repeat;
            background-position: right 10px center;
            background-size: 15px;
            padding-right: 30px;
        }
    }
}
</style>