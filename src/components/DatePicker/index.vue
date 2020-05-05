<template>
  <div
    class="datepicker__wrapper"
    v-if="show"
    v-on-click-outside="clickOutside"
    @blur="clickOutside"
  >
    <div
      class="datepicker__close-button -hide-on-desktop"
      v-if="isOpen"
      @click="hideDatepicker"
    >
      ＋
    </div>
    <div
      class="datepicker__dummy-wrapper"
      :class="{ 'datepicker__dummy-wrapper--is-active': isOpen }"
    >
      <date-input
        :i18n="i18n"
        :input-date="formatDate(checkIn)"
        input-date-type="check-in"
        :is-open="isOpen"
        :show-datepicker="showDatepicker"
        :toggle-datepicker="toggleDatepicker"
        :single-day-selection="singleDaySelection"
      />

      <date-input
        v-if="!singleDaySelection"
        :i18n="i18n"
        :input-date="formatDate(checkOut)"
        input-date-type="check-out"
        :is-open="isOpen"
        :showDatepicker="showDatepicker"
        :toggle-datepicker="toggleDatepicker"
        :single-day-selection="singleDaySelection"
      />
    </div>
    <div
      class="datepicker__clear-button"
      tabindex="0"
      @click="clearSelection"
      v-if="showClearSelectionButton"
    >
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 68 68">
        <path d="M6.5 6.5l55 55M61.5 6.5l-55 55"></path>
      </svg>
    </div>
    <div
      class="datepicker"
      :class="{ 'datepicker--open': isOpen, 'datepicker--closed': !isOpen }"
    >
      <div class="-hide-on-desktop">
        <div
          v-if="isOpen"
          class="datepicker__dummy-wrapper datepicker__dummy-wrapper--no-border"
          :class="{ 'datepicker__dummy-wrapper--is-active': isOpen }"
          @click="toggleDatepicker"
        >
          <div
            class="datepicker__input"
            tabindex="0"
            :class="{
              'datepicker__dummy-input--is-active': isOpen && checkIn == null
            }"
            type="button"
          >
            {{ `${checkIn ? formatDate(checkIn) : i18n["check-in"]}` }}
          </div>
          <div
            class="datepicker__input"
            tabindex="0"
            :class="{
              'datepicker__dummy-input--is-active':
                isOpen && checkOut == null && checkIn !== null
            }"
            type="button"
          >
            {{ `${checkOut ? formatDate(checkOut) : i18n["check-out"]}` }}
          </div>
        </div>
      </div>
      <div class="datepicker__inner">
        <div class="datepicker__header">
          <span
            class="datepicker__month-button datepicker__month-button--prev -hide-up-to-tablet"
            @click="renderPreviousMonth"
            @keyup.enter.stop.prevent="renderPreviousMonth"
            :tabindex="isOpen ? 0 : -1"
          ></span
          ><span
            class="datepicker__month-button datepicker__month-button--next -hide-up-to-tablet"
            @click="renderNextMonth"
            @keyup.enter.stop.prevent="renderNextMonth"
            :tabindex="isOpen ? 0 : -1"
          ></span>
        </div>
        <div class="datepicker__months" v-if="screenSize == 'desktop'">
          <div
            ref="datepickerMonth"
            class="datepicker__month"
            v-for="n in [0, 1]"
            :key="n"
          >
            <p class="datepicker__month-name">
              {{ getMonth(months[activeMonthIndex + n].days[15].date) }}
            </p>
            <div class="datepicker__week-row -hide-up-to-tablet">
              <div
                class="datepicker__week-name"
                v-for="(dayName, datePickerWeekIndex) in i18n['day-names']"
                :key="`datepicker__week-name-${datePickerWeekIndex}`"
              >
                {{ dayName }}
              </div>
            </div>
            <div
              class="square"
              v-for="(day, squareIndex) in months[activeMonthIndex + n].days"
              :key="`square-${squareIndex}`"
              @mouseover="hoveringDate = day.date"
            >
              <Day
                :activeMonthIndex="activeMonthIndex"
                :belongsToThisMonth="day.belongsToThisMonth"
                :checkIn="checkIn"
                :checkIncheckOutHalfDay="checkIncheckOutHalfDay"
                :checkOut="checkOut"
                :currentDateStyle="currentDateStyle"
                :date="day.date"
                :dayNumber="getDay(day.date)"
                :hoveringDate="hoveringDate"
                :is-open="isOpen"
                :nextDisabledDate="nextDisabledDate"
                :options="$props"
                :sortedDisabledDates="sortedDisabledDates"
                :tooltipMessage="tooltipMessage"
                @day-clicked="handleDayClick($event)"
              ></Day>
            </div>
          </div>
        </div>
        <div v-if="screenSize !== 'desktop' && isOpen">
          <div class="datepicker__week-row">
            <div
              class="datepicker__week-name"
              v-for="(dayName, datePickerIndex) in this.i18n['day-names']"
              :key="
                `datepicker__week-row-datepicker__week-name-${datePickerIndex}`
              "
            >
              {{ dayName }}
            </div>
          </div>
          <div
            class="datepicker__months"
            id="swiperWrapper"
            ref="swiperWrapper"
          >
            <div
              ref="datepickerMonth"
              class="datepicker__month"
              v-for="(a, n) in months"
              :key="n"
            >
              <p class="datepicker__month-name">
                {{ getMonth(months[n].days[15].date) }}
              </p>
              <div class="datepicker__week-row -hide-up-to-tablet">
                <div
                  class="datepicker__week-name"
                  v-for="(dayName, datePickerIndex) in i18n['day-names']"
                  :key="
                    `datepicker__month-name-datepicker__week-name-${datePickerIndex}`
                  "
                >
                  {{ dayName }}
                </div>
              </div>
              <div
                class="square"
                v-for="(day, index) in months[n].days"
                @mouseover="hoveringDate = day.date"
                @focus="hoveringDate = day.date"
                v-bind:key="index"
              >
                <Day
                  :activeMonthIndex="activeMonthIndex"
                  :belongsToThisMonth="day.belongsToThisMonth"
                  :checkIn="checkIn"
                  :checkIncheckOutHalfDay="checkIncheckOutHalfDay"
                  :checkOut="checkOut"
                  :currentDateStyle="currentDateStyle"
                  :date="day.date"
                  :dayNumber="getDay(day.date)"
                  :hoveringDate="hoveringDate"
                  :is-open="isOpen"
                  :nextDisabledDate="nextDisabledDate"
                  :options="$props"
                  :sortedDisabledDates="sortedDisabledDates"
                  :tooltipMessage="tooltipMessage"
                  @day-clicked="handleDayClick($event)"
                ></Day>
              </div>
            </div>
            <div
              class="next--mobile"
              @click="renderNextMonth"
              type="button"
            ></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import throttle from "lodash.throttle";
import { directive as onClickOutside } from "vue-on-click-outside";
import fecha from "fecha";

import Day from "../Day.vue";
import DateInput from "../DateInput.vue";
import Helpers from "../helpers";

const defaulti18n = {
  night: "Night",
  nights: "Nights",
  "day-names": ["Sun", "Mon", "Tue", "Wed", "Thur", "Fri", "Sat"],
  "check-in": "Check-in",
  "check-out": "Check-out",
  "month-names": [
    "January",
    "February",
    "March",
    "April",
    "May",
    "June",
    "July",
    "August",
    "September",
    "October",
    "November",
    "December"
  ]
};

export default {
  name: "HotelDatePicker",
  directives: {
    "on-click-outside": onClickOutside
  },
  components: {
    Day,
    DateInput
  },
  props: {
    currentDateStyle: {
      default: () => ({ border: "1px solid #00c690" })
    },
    value: {
      type: String
    },
    startingDateValue: {
      default: null,
      type: Date
    },
    endingDateValue: {
      default: null,
      type: Date
    },
    format: {
      default: "YYYY-MM-DD",
      type: String
    },
    startDate: {
      default() {
        return new Date();
      },
      type: [Date, String]
    },
    endDate: {
      default: Infinity,
      type: [Date, String, Number]
    },
    firstDayOfWeek: {
      default: 0,
      type: Number
    },
    minNights: {
      default: 1,
      type: Number
    },
    maxNights: {
      default: null,
      type: Number
    },
    halfDay: {
      type: Boolean,
      default: false
    },
    disabledDates: {
      default() {
        return [];
      },
      type: Array
    },
    disabledDaysOfWeek: {
      default() {
        return [];
      },
      type: Array
    },
    allowedRanges: {
      default() {
        return [];
      },
      type: Array
    },
    hoveringTooltip: {
      default: true,
      type: [Boolean, Function]
    },
    tooltipMessage: {
      default: null,
      type: String
    },
    i18n: {
      default: () => defaulti18n,
      type: Object
    },
    enableCheckout: {
      default: false,
      type: Boolean
    },
    singleDaySelection: {
      default: false,
      type: Boolean
    },
    showYear: {
      default: false,
      type: Boolean
    },
    closeDatepickerOnClickOutside: {
      default: true,
      type: Boolean
    },
    displayClearButton: {
      default: true,
      type: Boolean
    }
  },
  data() {
    return {
      hoveringDate: null,
      checkIn: this.startingDateValue,
      checkOut: this.endingDateValue,
      checkIncheckOutHalfDay: {},
      months: [],
      activeMonthIndex: 0,
      nextDisabledDate: null,
      show: true,
      isOpen: false,
      xDown: null,
      yDown: null,
      xUp: null,
      yUp: null,
      sortedDisabledDates: null,
      screenSize: null
    };
  },
  computed: {
    showClearSelectionButton() {
      return Boolean(
        (this.checkIn || this.checkOut) && this.displayClearButton
      );
    }
  },
  watch: {
    isOpen(value) {
      if (this.screenSize !== "desktop") {
        const body = document.querySelector("body");

        if (value) {
          body.style.overflow = "hidden";
          this.$nextTick(() => {
            if (this.$refs) {
              const { swiperWrapper } = this.$refs;
              const monthHeihgt = this.$refs.datepickerMonth[0].offsetHeight;

              swiperWrapper.scrollTop = this.activeMonthIndex * monthHeihgt;
            }
          });
        } else {
          body.style.overflow = "";
        }
      }
    },
    checkIn(newDate) {
      this.$emit("check-in-changed", newDate);
    },
    checkOut(newDate) {
      if (this.checkOut !== null && this.checkOut !== null) {
        this.hoveringDate = null;
        this.nextDisabledDate = null;
        this.show = true;
        this.parseDisabledDates();
        this.reRender();
        this.isOpen = false;
      }

      this.$emit("check-out-changed", newDate);
    }
  },
  beforeMount() {
    fecha.i18n = {
      dayNames: this.i18n["day-names"],
      dayNamesShort: this.shortenString(this.i18n["day-names"], 3),
      monthNames: this.i18n["month-names"],
      monthNamesShort: this.shortenString(this.i18n["month-names"], 3),
      amPm: ["am", "pm"],
      // D is the day of the month, function returns something like...  3rd or 11th
      DoFn(D) {
        return (
          D +
          ["th", "st", "nd", "rd"][
            D % 10 > 3 ? 0 : ((D - (D % 10) !== 10) * D) % 10
          ]
        );
      }
    };

    if (
      this.checkIn &&
      (this.getMonthDiff(
        this.getNextMonth(new Date(this.startDate)),
        this.checkIn
      ) > 0 ||
        this.getMonthDiff(this.startDate, this.checkIn) > 0)
    ) {
      this.createMonth(new Date(this.startDate));
      const count = this.getMonthDiff(this.startDate, this.checkIn);
      let nextMonth = new Date(this.startDate);

      for (let i = 0; i <= count; i++) {
        const tempNextMonth = this.getNextMonth(nextMonth);

        this.createMonth(tempNextMonth);
        nextMonth = tempNextMonth;
      }

      if (this.checkOut && this.getMonthDiff(this.checkIn, this.checkOut) > 0) {
        this.createMonth(this.getNextMonth(nextMonth));
        this.activeMonthIndex = 1;
      }

      this.activeMonthIndex += count;
    } else {
      this.createMonth(new Date(this.startDate));
      this.createMonth(this.getNextMonth(new Date(this.startDate)));
    }

    this.parseDisabledDates();
  },
  mounted() {
    this.handleWindowResize();

    if (this.screenSize !== "desktop") {
      document.addEventListener("touchstart", this.handleTouchStart, false);
      document.addEventListener("touchmove", this.handleTouchMove, false);
      document.addEventListener("touchend", this.handleTouchEnd, false);
    } else {
      window.addEventListener("resize", this.handleWindowResize);
    }

    this.onElementHeightChange(document.body, () => {
      this.emitHeighChangeEvent();
    });
  },
  destroyed() {
    if (this.screenSize !== "desktop") {
      document.removeEventListener("touchstart", this.handleTouchStart);
      document.removeEventListener("touchmove", this.handleTouchMove);
      document.removeEventListener("touchend", this.handleTouchEnd);
    } else {
      window.removeEventListener("resize", this.handleWindowResize);
    }
  },
  methods: {
    ...Helpers,
    formatDate(date) {
      if (date) {
        return fecha.format(date, this.format);
      }

      return "";
    },
    getDayDiff(d1, d2) {
      const t2 = new Date(d2).getTime();
      const t1 = new Date(d1).getTime();

      return parseInt((t2 - t1) / (24 * 3600 * 1000), 10);
    },
    handleWindowResize() {
      if (window.innerWidth < 480) {
        this.screenSize = "smartphone";
      } else if (window.innerWidth >= 480 && window.innerWidth < 768) {
        this.screenSize = "tablet";
      } else if (window.innerWidth >= 768) {
        this.screenSize = "desktop";
      }

      return this.screenSize;
    },
    onElementHeightChange(el, callback) {
      let lastHeight = el.clientHeight;
      let newHeight = lastHeight;
      const newEl = el;

      (function run() {
        newHeight = el.clientHeight;

        if (lastHeight !== newHeight) {
          callback();
        }

        lastHeight = newHeight;

        if (newEl.onElementHeightChangeTimer) {
          clearTimeout(el.onElementHeightChangeTimer);
        }

        newEl.onElementHeightChangeTimer = setTimeout(run, 1000);
      })();
    },
    emitHeighChangeEvent() {
      this.$emit("height-changed");
    },
    reRender() {
      this.show = false;
      this.$nextTick(() => {
        this.show = true;
      });
    },
    clearSelection() {
      this.hoveringDate = null;
      this.checkIn = null;
      this.checkOut = null;
      this.nextDisabledDate = null;
      this.show = true;
      this.parseDisabledDates();
      this.reRender();
      this.$emit("clear-selection");
    },
    hideDatepicker() {
      this.isOpen = false;
    },
    showDatepicker() {
      this.isOpen = true;
    },
    toggleDatepicker() {
      this.isOpen = !this.isOpen;
    },
    clickOutside(e) {
      // hide datePicker when event is outside
      if (
        this.closeDatepickerOnClickOutside &&
        e.target.dataset.qa !== "datepickerInput"
      ) {
        this.hideDatepicker();
      }
    },
    handleDayClick(event) {
      if (this.checkIn == null && this.singleDaySelection === false) {
        this.checkIn = event.date;
      } else if (this.singleDaySelection === true) {
        this.checkIn = event.date;
        this.checkOut = event.date;
      } else if (this.checkIn !== null && this.checkOut == null) {
        this.checkOut = event.date;
      } else {
        this.checkOut = null;
        this.checkIn = event.date;
      }

      this.nextDisabledDate = event.nextDisabledDate;
    },
    renderPreviousMonth() {
      if (this.activeMonthIndex >= 1) {
        this.activeMonthIndex--;
      }
    },
    renderNextMonth: throttle(function throttleRenderNextMonth() {
      if (this.activeMonthIndex < this.months.length - 2) {
        this.activeMonthIndex++;

        return;
      }

      let firstDayOfLastMonth;

      if (this.screenSize !== "desktop") {
        firstDayOfLastMonth = this.months[this.months.length - 1].days.filter(
          day => day.belongsToThisMonth === true
        );
      } else {
        firstDayOfLastMonth = this.months[
          this.activeMonthIndex + 1
        ].days.filter(day => day.belongsToThisMonth === true);
      }

      if (this.endDate !== Infinity) {
        if (
          fecha.format(firstDayOfLastMonth[0].date, "YYYYMM") ===
          fecha.format(new Date(this.endDate), "YYYYMM")
        ) {
          return;
        }
      }

      this.createMonth(this.getNextMonth(firstDayOfLastMonth[0].date));

      this.activeMonthIndex++;
    }, 200),
    setCheckIn(date) {
      this.checkIn = date;
    },
    setCheckOut(date) {
      this.checkOut = date;
    },
    getDay(date) {
      return fecha.format(date, "D");
    },
    getMonth(date) {
      return (
        this.i18n["month-names"][fecha.format(date, "M") - 1] +
        (this.showYear ? fecha.format(date, " YYYY") : "")
      );
    },
    createMonth(date) {
      const firstDay = this.getFirstDay(date, this.firstDayOfWeek);
      const month = {
        days: []
      };

      for (let i = 0; i < 42; i++) {
        month.days.push({
          date: this.addDays(firstDay, i),
          belongsToThisMonth:
            this.addDays(firstDay, i).getMonth() === date.getMonth(),
          isInRange: false
        });
      }

      this.months.push(month);
    },
    parseDisabledDates() {
      let sortedDates = [];
      const checkIncheckOutHalfDay = {};

      for (let i = 0; i < this.disabledDates.length; i++) {
        const newDate = this.disabledDates[i];

        if (this.halfDay) {
          const newDateIncrementOne = this.disabledDates[i + 1];

          if (i === 0) {
            checkIncheckOutHalfDay[newDate] = {
              checkIn: true
            };
          }

          if (
            this.disabledDates[i + 1] &&
            this.getDayDiff(newDate, newDateIncrementOne) > 1
          ) {
            checkIncheckOutHalfDay[newDate] = {
              checkOut: true
            };
            checkIncheckOutHalfDay[newDateIncrementOne] = {
              checkIn: true
            };
          }

          if (i === this.disabledDates.length - 1) {
            checkIncheckOutHalfDay[
              this.disabledDates[this.disabledDates.length - 1]
            ] = {
              checkOut: true
            };
          }
        }

        sortedDates[i] = this.disabledDates[i];
      }

      if (this.halfDay) {
        const halfDays = Object.keys(checkIncheckOutHalfDay);

        sortedDates = sortedDates.filter(date => !halfDays.includes(date));
        sortedDates.map(date => new Date(date));
      }

      sortedDates.sort((a, b) => a - b);

      this.checkIncheckOutHalfDay = checkIncheckOutHalfDay;
      this.sortedDisabledDates = sortedDates;
    }
  }
};
</script>

<style lang="scss" src="./index.scss"></style>