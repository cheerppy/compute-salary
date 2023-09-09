<script setup lang="ts">
// meta information
useHead({
    title: "給与概算",
    meta: [
        { name: "description", content: "給与を概算する。" }
    ],
    htmlAttrs: { lang: "ja" }
})

// utils
function hour2Time(val: number) {
    if (!val) { return "" }
    const h = `${~~val}`.padStart(2, "0")
    const min = `${~~((val - +h) * 60)}`.padStart(2, "0")
    return h + ":" + min
}
function time2Hour(val: string) {
    if (!val) { return 0 }
    const [h, min] = val.split(":")
    return +h + +min / 60
}

function hr(time: Ref<string>) {
    return time2Hour(time.value)
}

// 基本給
const d__base = ref(200000)
const base = computed({ get() { { return d__base.value || "" } }, set(val) { d__base.value = +val } })

// 所定労働日数
const d__day = ref(20)
const day = computed({ get() { { return d__day.value || "" } }, set(val) { d__day.value = +val } })


// 時給
const d__hourly = ref(1325)
const hourly = computed({ get() { { return d__hourly.value || "" } }, set(val) { d__hourly.value = +val } })
function resetHourly() {
    const rawHourly = d__base.value / d__day.value / 8
    d__hourly.value = ~~(rawHourly * 1000) / 1000 // 小数3桁以下を切り捨てる
}

// 残業
const d__fixedOvertime = ref("00:00") // 法定内残業時間（半休時の残業など）
const d__overtime = ref("20:00") // 法定外残業時間
const fixedOvertime = computed({ get() { { return d__fixedOvertime.value } }, set(val) { d__fixedOvertime.value = val } })
const overtime = computed({ get() { { return d__overtime.value || "" } }, set(val) { d__overtime.value = val || "" } })
const overtimePay = computed(() => ~~(hr(d__overtime) * 1.25 + hr(d__fixedOvertime)) * d__hourly.value)
function onFocus(event: Event) {
    // focusされたとき、テキスト全体を選択状態にする
    // focusされている間のみ、残業時間を2桁入力したら自動で":"を挿入する
    const input = event.target! as HTMLInputElement
    const unwatch = watch(overtime, (time) => {
        if (time.length === 2) { overtime.value = time + ":" }
    })
    input.focus()
    input.select()
    input.addEventListener("blur", unwatch)
}

// 深夜
const d__midnightTime = ref("00:00")
const midnightTime = computed({ get() { return d__midnightTime.value }, set(val) { d__midnightTime.value = val } })

// 不就労
const d__unemployed = ref("00:00")
const unemployed = computed({ get() { return d__unemployed.value }, set(val) { d__unemployed.value = val } })

// 所得
const grossIncome = computed(() => d__base.value + /* 調整手当 */10000 + (hr(d__overtime) * 1.25 + hr(d__midnightTime) * 0.5625 + hr(d__fixedOvertime) - hr(d__unemployed)) * d__hourly.value)

// 控除
const socialInsurance = computed(() => /* 健康保険料 */10010 + /* 厚生年金保険料 */20130 + /* 雇用保険料 */(grossIncome.value * 0.006))
const otherDeductions = /* 寮費 */28500 + /* その他 */630 + /* 確定拠出年金 */1000
const residenceTax = 5900
const incomeTax = computed(() => grossIncome.value * 0.02)
const totalDeductions = computed(() => socialInsurance.value + otherDeductions + residenceTax + incomeTax.value)


// 給与
const deductedIncome = computed(() => grossIncome.value - totalDeductions.value + /* 交通費 */6120)

</script>

<template>
    <div class="wrapper">
        <div class="container">
            <table class="grid-2">
                <caption>契約内容</caption>
                <tr>
                    <td>基本給</td>
                    <td><input type="number" id="base" v-model="base" step="1000" size="8">円</td>
                </tr>
                <tr>
                    <td>所定労働日数</td>
                    <td><input type="number" id="day" v-model="day" size="3">日</td>
                </tr>
                <tr>
                    <td>時給 <button @click="resetHourly">計算</button> </td>
                    <td> <input type="number" id="hourly" v-model="hourly" step="50" size="8">円/h</td>
                </tr>
            </table>
            <table class="grid-2">
                <caption>勤務実績</caption>
                <tr>
                    <td>残業時間</td>
                    <td> <input type="text" mode="numeric" id="overtime" v-model="overtime" step="600" size="5"
                            @focus="onFocus">h </td>
                </tr>
                <tr>
                    <td>（法定内残業時間</td>
                    <td> <input type="time" id="fixed-overtime" v-model="fixedOvertime" step="600">h ）</td>
                </tr>
                <tr>
                    <td>深夜時間</td>
                    <td> <input type="time" id="midnight" v-model="midnightTime" step="600">h </td>
                </tr>
                <tr>
                    <td>不就労</td>
                    <td> <input type="time" id="unemployed" v-model="unemployed">h </td>
                </tr>
            </table>
            <div class="br"></div>
            <table class="grid-2">
                <caption>給与/所得</caption>
                <tr>
                    <td>残業代</td>
                    <td> <input type="text" class="salary" size="8" id="overtime_pay"
                            :value="(~~overtimePay).toLocaleString()" readonly>円
                    </td>
                </tr>
                <tr>
                    <td>所得</td>
                    <td> <input type="text" class="salary" size="8" id="total" :value="(~~grossIncome).toLocaleString()"
                            readonly>円
                    </td>
                </tr>
                <tr class="deducted-income">
                    <td>給与</td>
                    <td> <input type="text" class="salary" size="8" id="deducted-income"
                            :value="(~~deductedIncome).toLocaleString()" readonly>円
                    </td>
                </tr>
            </table>
            <!-- <div>
                <table>
                    <caption>utility</caption>
                    <tr>
                        <td>時間</td>
                        <td><input type="time" id=""></td>
                    </tr>
                </table>
            </div> -->
        </div>
    </div>
</template>

<style>
.wrapper {
    display: flex;
    justify-content: center;
}

.container {
    min-width: 300px;
    width: 80%;
    display: flex;
    flex-wrap: wrap;
    flex-shrink: 0;
    justify-content: center;
    gap: 10px 20px;
}

div.br {
    width: 100%;
    margin: 0;
}

.tables_1 {
    display: flex;
    flex-shrink: 0;
    justify-content: space-between;
    gap: 20px;
}

table {
    display: grid;
    column-gap: 20px;
    row-gap: 5px;
    justify-content: center;
}

table caption {
    display: block;
    grid-area: 1 / 1 / 2 / 3;
}

table.grid-2 {
    grid-template-columns: repeat(2, auto);
}

table tr {
    display: contents;
}

table td {
    display: inline-block;
}

table,
*.tr,
*.td {
    margin: 0;
}

input {
    width: 6em;
}

input#deducted-income {
    background-color: yellow;
    font-weight: bold;
}

.deducted-income {
    font-weight: bold;
}

.salary {
    text-align: right;
}
</style>