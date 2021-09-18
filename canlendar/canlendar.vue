<!--
 * @Author: wuxx
 * @Date: 2021-06-02 16:27:01
 * @LastEditTime: 2021-07-01 14:27:22
 * @LastEditors: wuxx
 * @Description:
 * @FilePath: \jskhpj\components\canlendar.vue
-->
<template>
  <div class="outwrap">
    <div v-if="daystype !== 'by_week'" class="top">
      <ul class="top__content">
        <li class="top__content__one">
          <div>
            <a-icon @click="topmoveto('left')" type="left" />
          </div>
          <div style="width:180px;text-align:center">
            <transition-group
              :leave-to-class="topmoveClass"
              :enter-class="topmoveClass"
              leave-class="leaveClass"
              enter-active-class="topenterActive"
              name="label"
              tag="span"
            >
              <span :key="currentYear" style="display:inline-block">{{
                currentYear
              }}</span>
            </transition-group>
          </div>
          <div>
            <a-icon @click="topmoveto('right')" type="right" />
          </div>
        </li>
        <li
          v-if="daystype === 'by_day' || daystype === 'by_month'"
          @click="backToToday"
          :style="{ right: daystype === 'by_day' ? '48px' : '0' }"
          class="top__content__two"
        >
          {{ daystype === 'by_day' ? '回到今天' : '回到本月' }}
        </li>
        <li
          v-if="daystype === 'by_day'"
          @click="show = !show"
          class="top__content__three"
        >
          {{ show ? '收起' : '展开' }}
        </li>
      </ul>
    </div>
    <div v-show="!show" class="bottom">
      <div @click="moveto('left')" class="bottom__left">
        <a-icon type="left" />
      </div>
      <div class="bottom__content">
        <div class="bottom__content__wrap">
          <transition-group
            :leave-to-class="moveClass"
            :enter-class="moveClass"
            leave-class="leaveClass"
            enter-active-class="enterActive"
            name="list"
            tag="div"
          >
            <span
              v-for="i in daysArray"
              :key="i.day"
              :class="{
                chosen: ischosen == i.day,
                today:
                  (currentDay == i.day &&
                    daystype !== 'by_week' &&
                    currentDay.slice(0, 4) == currentYear.slice(0, 4)) ||
                  (currentDay == i.day && daystype === 'by_week'),
                fontcolor: i.isFuture === '1'
              }"
              @click="getchosen(i.day, i.isFuture)"
              class="btn"
            >
              <a-badge>
                <a-icon
                  slot="count"
                  v-if="
                    ischosen !== i.day &&
                      currentDay !== i.day &&
                      i.hasRecord === '1'
                  "
                  :style="{
                    color: '#16BC84',
                    'background-color': '#fff',
                    'border-radius': '7px'
                  }"
                  type="check-circle"
                  theme="filled"
                />
                <span v-if="daystype === 'by_month'">{{
                  i.day.slice(5, 7) + '月'
                }}</span>
                <span v-if="daystype === 'by_day'">{{
                  i.day.slice(5, 7) + '月' + i.day.slice(8, 10) + '日'
                }}</span>
                <span v-if="daystype === 'by_week'">
                  {{ '第' + i.day + '周' }}
                </span>
              </a-badge>
            </span>
          </transition-group>
        </div>
      </div>
      <div @click="moveto('right')" class="bottom__right">
        <a-icon type="right" />
      </div>
    </div>

    <transition name="slide-fade">
      <div v-if="show" class="dates">
        <ul class="dates__week">
          <li>周日</li>
          <li>周一</li>
          <li>周二</li>
          <li>周三</li>
          <li>周四</li>
          <li>周五</li>
          <li>周六</li>
        </ul>
        <div
          v-for="(i, index) of monthDaysBySeven"
          :key="index"
          class="dates__days"
        >
          <div
            v-for="(p, s) of i"
            :key="s"
            :class="{
              singlechose: ischosen == p.day && p.day,
              todaychoise: currentDay == p.day && p.day,
              singlefontcolor: p.isFuture === '1'
            }"
            @click="choseDate(p.day, p.isFuture)"
            class="dates__days__singleday"
          >
            <a-badge>
              <a-icon
                slot="count"
                v-if="
                  ischosen !== p.day &&
                    currentDay !== p.day &&
                    p.hasRecord === '1'
                "
                :style="{
                  color: '#16BC84',
                  'background-color': '#fff',
                  'border-radius': '7px'
                }"
                type="check-circle"
                theme="filled"
              />
              <span>{{ p.day.slice(8, 10) }}</span>
            </a-badge>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>
<script>
export default {
  props: {
    dataSource: {
      type: Array,
      default: () => []
    },
    daystype: {
      type: String,
      require: true,
      default: 'by_day'
    }
  },
  data() {
    return {
      ischosen: '',
      direction: '',
      show: false,
      currentDay: '', // 今天
      moveClass: 'enterClass',
      topmoveClass: 'topenterClass',
      currentYear: '', // 当前年或年月
      allDaysArr: [], // 所有的年月里的日子
      daysArray: [], // 单个年月里的日子
      currentIndex: 1, // 计算第几个7
      sevens: 0, // 当前月份有几个7
      currentYearIndex: 0, // 当前年的index
      currentYearLength: 1, // 年份的长度
      currentMonthDays: [], // 当前月的全部天
      monthDaysBySeven: [],
      splitSevenNum: 0
    }
  },
  watch: {
    dataSource: {
      handler(val) {
        if (!val || !val.length) return
        const temparr = []
        this.allDaysArr = []
        this.currentIndex = 1
        this.ischosen = ''
        this.currentDay = ''
        this.currentMonthDays = []
        this.monthDaysBySeven = []
        this.currentYearIndex = 0
        this.sevens = 0
        this.show = false
        // 如果不存在今天，则默认选中第一条数据
        const isHasToday = val.some(item => item.nowDate === '1')
        if (!isHasToday) {
          this.ischosen = val[0].date
        }
        if (this.daystype === 'by_day') {
          if (val.length === 1) {
            const t = val[0].date
            this.allDaysArr = [
              {
                year: t.slice(0, 4),
                month: t.slice(5, 7),
                days: [
                  {
                    day: t,
                    week: val[0].weekDay,
                    nowDate: val[0].nowDate,
                    hasRecord: val[0].hasRecord,
                    isFuture: val[0].isFuture
                  }
                ]
              }
            ]
            if (val[0].nowDate === '1') {
              this.ischosen = t
              this.currentDay = t
            }
          } else {
            val.reduce((pre, cur, curindex) => {
              const preyear = pre.date.slice(0, 4)
              const premonth = pre.date.slice(5, 7)
              const preday = pre.date
              const preweek = pre.weekDay
              const nowDate = pre.nowDate
              const hasRecord = pre.hasRecord
              const isFuture = pre.isFuture
              if (nowDate === '1') {
                this.ischosen = preday
                this.currentDay = preday
              }
              if (val.length === 2) {
                temparr.push({
                  year: preyear,
                  data: [
                    {
                      year: preyear,
                      month: premonth,
                      days: [
                        {
                          day: preday,
                          week: preweek,
                          nowDate,
                          hasRecord,
                          isFuture
                        }
                      ]
                    }
                  ]
                })
                if (cur.nowDate === '1') {
                  this.ischosen = cur.date
                  this.currentDay = cur.date
                }
                this.getdays(
                  temparr,
                  cur.date.slice(0, 4),
                  cur.date.slice(5, 7),
                  cur.date,
                  cur.weekDay,
                  cur.nowDate,
                  cur.hasRecord,
                  cur.isFuture
                )
              } else if (curindex === 1) {
                temparr.push({
                  year: preyear,
                  data: [
                    {
                      year: preyear,
                      month: premonth,
                      days: [
                        {
                          day: preday,
                          week: preweek,
                          nowDate,
                          hasRecord,
                          isFuture
                        }
                      ]
                    }
                  ]
                })
              } else if (curindex === val.length - 1) {
                this.getdays(
                  temparr,
                  preyear,
                  premonth,
                  preday,
                  preweek,
                  nowDate,
                  hasRecord,
                  isFuture
                )
                if (cur.nowDate === '1') {
                  this.ischosen = cur.date
                  this.currentDay = cur.date
                }
                this.getdays(
                  temparr,
                  cur.date.slice(0, 4),
                  cur.date.slice(5, 7),
                  cur.date,
                  cur.weekDay,
                  cur.nowDate,
                  cur.hasRecord,
                  cur.isFuture
                )
              } else {
                this.getdays(
                  temparr,
                  preyear,
                  premonth,
                  preday,
                  preweek,
                  nowDate,
                  hasRecord,
                  isFuture
                )
              }
              return cur
            })
            temparr.forEach(item => {
              this.allDaysArr.push(...item.data)
            })
          }
          if (isHasToday) {
            this.backToToday()
          } else {
            this.daysArray = this.allDaysArr[0].days.slice(0, 7)
            this.currentYear =
              this.allDaysArr[0].year + '年' + this.allDaysArr[0].month + '月'
            this.sevens = Math.ceil(this.allDaysArr[0].days.length / 7)
            this.refreshDays(this.allDaysArr[0])
          }
          this.currentYearLength = this.allDaysArr.length
        } else if (this.daystype === 'by_week') {
          val.forEach(item => {
            temparr.push({
              day: item.date,
              nowDate: item.nowDate,
              hasRecord: item.hasRecord,
              isFuture: item.isFuture
            })
            if (item.nowDate === '1') {
              this.ischosen = item.date
              this.currentDay = item.date
            }
          })
          this.allDaysArr = temparr
          if (isHasToday) {
            this.allDaysArr.filter((item, index) => {
              if (item.nowDate === '1') {
                const num = Math.ceil((index + 1) / 7)
                this.currentIndex = num
                this.daysArray = this.allDaysArr.slice(7 * num - 7, 7 * num)
                return true
              }
            })
          } else {
            this.daysArray = temparr.slice(0, 7)
          }
          this.sevens = Math.ceil(temparr.length / 7)
        } else if (this.daystype === 'by_month') {
          if (val.length === 1) {
            this.allDaysArr = [
              {
                year: val[0].date.slice(0, 4),
                days: [
                  {
                    day: val[0].date,
                    nowDate: val[0].nowDate,
                    hasRecord: val[0].hasRecord,
                    isFuture: val[0].isFuture
                  }
                ]
              }
            ]
            if (val[0].nowDate === '1') {
              this.ischosen = val[0].date
              this.currentDay = val[0].date
            }
          } else {
            val.reduce((pre, cur, curindex) => {
              const preyear = pre.date.slice(0, 4)
              const premonth = pre.date
              const nowDate = pre.nowDate
              const hasRecord = pre.hasRecord
              const isFuture = pre.isFuture
              if (nowDate === '1') {
                this.ischosen = pre.date
                this.currentDay = pre.date
              }
              if (val.length === 2) {
                temparr.push({
                  year: preyear,
                  days: [
                    {
                      day: pre.date,
                      nowDate,
                      hasRecord,
                      isFuture
                    }
                  ]
                })
                if (cur.nowDate === '1') {
                  this.ischosen = cur.date
                  this.currentDay = cur.date
                }
                this.getMonths(
                  temparr,
                  cur.date.slice(0, 4),
                  cur.date,
                  cur.nowDate,
                  cur.hasRecord,
                  cur.isFuture
                )
              } else if (curindex === 1) {
                temparr.push({
                  year: preyear,
                  days: [
                    {
                      day: pre.date,
                      nowDate,
                      hasRecord,
                      isFuture
                    }
                  ]
                })
              } else if (curindex === val.length - 1) {
                this.getMonths(
                  temparr,
                  preyear,
                  premonth,
                  nowDate,
                  hasRecord,
                  isFuture
                )
                if (cur.nowDate === '1') {
                  this.ischosen = cur.date
                  this.currentDay = cur.date
                }
                this.getMonths(
                  temparr,
                  cur.date.slice(0, 4),
                  cur.date,
                  cur.nowDate,
                  cur.hasRecord,
                  cur.isFuture
                )
              } else {
                this.getMonths(
                  temparr,
                  preyear,
                  premonth,
                  nowDate,
                  hasRecord,
                  isFuture
                )
              }
              return cur
            })
            this.allDaysArr = temparr
          }
          if (isHasToday) {
            this.backToToday()
          } else {
            this.daysArray = this.allDaysArr[0].days.slice(0, 7)
            this.currentYear = this.allDaysArr[0].year + '年'
            this.sevens = Math.ceil(this.allDaysArr[0].days.length / 7)
          }
          this.currentYearLength = this.allDaysArr.length
        }
      },
      deep: true,
      immediate: true
    },
    ischosen: {
      handler(val) {
        if (!val) return
        this.$emit('chosen', val)
      },
      immediate: true
    }
  },
  methods: {
    getdays(
      temparr,
      preyear,
      premonth,
      preday,
      preweek,
      nowDate,
      hasRecord,
      isFuture
    ) {
      // 检测数组中是否存在该年份
      const isYearExist = temparr.some(item => item.year === preyear)
      if (isYearExist) {
        temparr.some(item => {
          if (item.year === preyear) {
            // 检测该年份的月份是否存在
            const isMonthExist = item.data.some(mon => mon.month === premonth)
            if (isMonthExist) {
              item.data.some(mon => {
                // 在相对应的月份中添加天数
                if (mon.month === premonth) {
                  mon.days.push({
                    day: preday,
                    week: preweek,
                    nowDate,
                    hasRecord,
                    isFuture
                  })
                  return true
                }
              })
            } else {
              // 添加没有的月份
              item.data.push({
                year: preyear,
                month: premonth,
                days: [
                  {
                    day: preday,
                    week: preweek,
                    nowDate,
                    hasRecord,
                    isFuture
                  }
                ]
              })
            }
          }
        })
      } else {
        // 添加没有的年份对象
        temparr.push({
          year: preyear,
          data: [
            {
              year: preyear,
              month: premonth,
              days: [
                {
                  // day: premonth + '月' + preday + '日',
                  day: preday,
                  week: preweek,
                  nowDate,
                  hasRecord,
                  isFuture
                }
              ]
            }
          ]
        })
      }
    },
    getMonths(temparr, preyear, premonth, nowDate, hasRecord, isFuture) {
      // 检测数组中是否存在该年份
      const isYearExist = temparr.some(item => item.year === preyear)
      if (isYearExist) {
        temparr.some(item => {
          if (item.year === preyear) {
            // 添加没有的月份
            item.days.push({
              day: premonth,
              nowDate,
              hasRecord,
              isFuture
            })
          }
        })
      } else {
        // 添加没有的年份对象
        temparr.push({
          year: preyear,
          days: [{ day: premonth, nowDate, hasRecord, isFuture }]
        })
      }
    },
    // 补充不够的天数
    dealMonthDays(arr) {
      const nextweek = arr[0].week
      if (arr[0].week === '7') return
      if (nextweek !== '1') {
        arr.unshift({ week: (Number(nextweek) - 1).toString(), day: '' })
      } else {
        arr.unshift({ week: '7', day: '' })
      }
      this.dealMonthDays(arr)
    },
    // 月份里的天数分成7个一份
    splitBySeven(arr) {
      if (arr.length >= 7) {
        arr.some((item, index) => {
          if (index <= 6) {
            if (!this.monthDaysBySeven[this.splitSevenNum]) {
              this.monthDaysBySeven[this.splitSevenNum] = []
            }
            this.monthDaysBySeven[this.splitSevenNum].push(item)
          } else {
            return true
          }
        })
        this.splitSevenNum++
        arr.splice(0, 7)
        this.splitBySeven(arr)
      } else {
        arr.forEach(item => {
          if (!this.monthDaysBySeven[this.splitSevenNum]) {
            this.monthDaysBySeven[this.splitSevenNum] = []
          }
          this.monthDaysBySeven[this.splitSevenNum].push(item)
        })
      }
    },
    getchosen(day, isFuture) {
      if (isFuture === '1') return
      this.ischosen = day
    },
    choseDate(day, isFuture) {
      if (!day || isFuture === '1') return
      this.ischosen = day
      this.fastChange(day)
      this.show = false
    },
    backToToday() {
      this.ischosen = this.currentDay
      // 按月
      if (this.daystype === 'by_month') {
        this.allDaysArr.filter((item, index) => {
          if (item.year === this.currentDay.slice(0, 4)) {
            this.currentYear = item.year + '年'
            this.currentYearIndex = index
            return true
          }
        })
      } else if (this.daystype === 'by_day') {
        // 按天
        this.allDaysArr.filter((item, index) => {
          if (
            item.year === this.currentDay.slice(0, 4) &&
            item.month === this.currentDay.slice(5, 7)
          ) {
            this.currentYear = item.year + '年' + item.month + '月'
            this.currentYearIndex = index
            this.refreshDays(item)
            return true
          }
        })
      }
      this.fastChange(this.currentDay)
    },
    // 日子或周的切换
    moveto(dir) {
      this.direction = dir
      if (dir === 'left' && this.currentIndex > 1) {
        this.moveClass = 'enterClass'
        this.currentIndex--
      } else if (dir === 'right' && this.currentIndex < this.sevens) {
        this.moveClass = 'backenterClass'
        this.currentIndex++
      }
      if (this.currentIndex > 1 || this.currentIndex < this.sevens) {
        if (this.daystype === 'by_day' || this.daystype === 'by_month') {
          this.sevenChange(this.allDaysArr[this.currentYearIndex].days)
        } else if (this.daystype === 'by_week') {
          this.sevenChange(this.allDaysArr)
        }
      }
    },
    // 年月的切换
    topmoveto(dir) {
      if (dir === 'left' && this.currentYearIndex > 0) {
        this.currentYearIndex--
        this.topmoveClass = 'topenterClass'
      } else if (
        dir === 'right' &&
        this.currentYearIndex < this.currentYearLength - 1
      ) {
        this.currentYearIndex++
        this.topmoveClass = 'topbackenterClass'
      }
      if (
        this.currentYearIndex > 0 ||
        this.currentYearIndex < this.currentYearLength - 1
      ) {
        if (this.daystype === 'by_day') {
          this.currentYear =
            this.allDaysArr[this.currentYearIndex].year +
            '年' +
            this.allDaysArr[this.currentYearIndex].month +
            '月'
          // 刷新当前月份的天
          this.refreshDays(this.allDaysArr[this.currentYearIndex])
        } else if (this.daystype === 'by_month') {
          this.currentYear = this.allDaysArr[this.currentYearIndex].year + '年'
          this.sevens = Math.ceil(
            this.allDaysArr[this.currentYearIndex].days.length / 7
          )
        }
        this.daysArray = this.allDaysArr[this.currentYearIndex].days.slice(0, 7)
        this.currentIndex = 1
      }
    },
    // 刷新当前月份的天数
    refreshDays(arr) {
      this.monthDaysBySeven = []
      this.sevens = Math.ceil(arr.days.length / 7)
      this.currentMonthDays = arr.days.map(item => item)
      this.dealMonthDays(this.currentMonthDays)
      this.splitSevenNum = 0
      this.splitBySeven(this.currentMonthDays)
    },
    // 快速切换时间
    fastChange(target) {
      this.allDaysArr[this.currentYearIndex].days.filter((item, index) => {
        if (item.day === target) {
          this.currentIndex = Math.ceil((index + 1) / 7)
          this.sevenChange(this.allDaysArr[this.currentYearIndex].days)
          return true
        }
      })
    },
    // 切7
    sevenChange(arr) {
      this.daysArray = arr.slice(
        7 * this.currentIndex - 7,
        7 * this.currentIndex
      )
    }
  }
}
</script>
<style lang="scss" scoped>
.outwrap {
  width: 100%;
  background-color: #fff;
  padding: 0 20px 0 20px;
  position: relative;
  margin-bottom: 16px;
  .top {
    height: 57px;
    border-bottom: 1px solid #eeeeee;
    &__content {
      height: 100%;
      position: relative;
      color: #222;
      &__one {
        line-height: 57px;
        position: absolute;
        font-size: 18px;
        left: 50%;
        transform: translateX(-50%);
        display: flex;
      }
      &__two {
        cursor: pointer;
        position: absolute;
        // right: 48px;
        font-size: 12px;
        line-height: 24px;
        height: 24px;
        border: 1px solid #cccccc;
        border-radius: 16px;
        padding: 0 8px 0 8px;
        margin-right: 8px;
        margin-left: 230px;
        top: 50%;
        transform: translateY(-50%);
      }
      &__three {
        right: 0;
        position: absolute;
        font-size: 12px;
        line-height: 24px;
        height: 24px;
        border: 1px solid #cccccc;
        border-radius: 16px;
        padding: 0 8px 0 8px;
        cursor: pointer;
        top: 50%;
        transform: translateY(-50%);
      }
    }
  }
  .bottom {
    height: 51px;
    width: 100%;
    border-bottom: 1px solid #eeeeee;
    color: #222;
    position: relative;
    overflow: hidden;
    &__left {
      background-color: #fff;
      cursor: pointer;
      position: absolute;
      height: 100%;
      left: 0;
      padding-top: 15px;
      z-index: 10;
    }
    .bottom__content {
      position: absolute;
      left: 14px;
      height: 100%;
      width: calc(100% - 28px);
      text-align: center;
      font-size: 14px;
      // transition: transform 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
      &__wrap {
        position: absolute;
        height: 100%;
        width: 100%;
        left: 0;
        display: flex;
        align-items: center;
        flex-wrap: nowrap;
        justify-content: center;
        .btn {
          display: inline-block;
          text-align: center;
          min-width: 82px;
          margin: 0 15px 0 15px;
          cursor: pointer;
        }
        .today {
          display: inline-block;
          text-align: center;
          min-width: 82px;
          color: #005eff;
          background-color: #e5eeff;
          padding: 6px 15px 6px 15px;
          border-radius: 16px;
          margin: 0 15px 0 15px;
        }
        .chosen {
          display: inline-block;
          text-align: center;
          min-width: 82px;
          color: #fff;
          background-color: #005eff;
          padding: 6px 15px 6px 15px;
          border-radius: 16px;
          margin: 0 15px 0 15px;
        }
        .fontcolor {
          color: #cccccc;
        }
      }
    }
    &__right {
      background-color: #fff;
      position: absolute;
      cursor: pointer;
      height: 100%;
      padding-top: 15px;
      right: 0;
      z-index: 10;
    }
  }
  .dates {
    position: absolute;
    left: 0;
    z-index: 20;
    width: 100%;
    padding: 0 20px 10px 20px;
    background-color: #fff;
    border-bottom-left-radius: 4px;
    border-bottom-right-radius: 4px;
    box-shadow: 0px 8px 8px 0 rgba(0, 0, 0, 0.1);
    &__week {
      height: 51px;
      display: flex;
      // justify-content: space-between;
      justify-content: flex-start;
      align-items: center;
      font-size: 16px;
      color: #888888;
      border-bottom: 1px solid #eeeeee;
      padding: 0 72px 0 72px;
      li {
        min-width: calc(100% / 7);
        text-align: center;
      }
    }
    &__days {
      padding: 0 72px 0 72px;
      display: flex;
      justify-content: flex-start;
      flex-wrap: wrap;
      &__singleday {
        min-width: calc(100% / 7 - 40px);
        text-align: center;
        line-height: 60px;
        font-size: 16px;
        margin: 0 20px 0 20px;
        // padding: 0 18px 0 0;
        color: #222222;
        cursor: pointer;
      }
      .todaychoise {
        background: #e5eeff;
        margin: 14px 20px 14px 20px;
        padding: 0 20px 0 20px;
        color: #005eff;
        border-radius: 16px;
        font-size: 16px;
        line-height: 32px;
      }
      .singlechose {
        background: #005eff;
        margin: 14px 20px 14px 20px;
        padding: 0 20px 0 20px;
        color: #fff;
        border-radius: 16px;
        font-size: 16px;
        line-height: 32px;
      }
      .singlefontcolor {
        color: #cccccc;
      }
    }
  }
  .slide-fade-enter-active {
    transition: all 0.3s ease;
  }
  .slide-fade-leave-active {
    transition: all 0.3s ease;
  }
  .slide-fade-enter,
  .slide-fade-leave-to {
    transform: translateY(-20px);
    opacity: 0;
  }
  .enterActive {
    transition: all 0.5s ease;
  }
  .leaveClass {
    opacity: 0;
  }
  .enterClass {
    opacity: 0;
    transform: translateX(-150px);
  }
  .backenterClass {
    opacity: 0;
    transform: translateX(150px);
  }
  .topenterActive {
    transition: all 0.5s cubic-bezier(1, 0.5, 0.8, 1);
  }
  .topenterClass {
    opacity: 0;
    transform: translateX(-10px);
  }
  .topbackenterClass {
    opacity: 0;
    transform: translateX(10px);
  }
}
</style>
