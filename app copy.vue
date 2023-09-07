<!-- backup -->
<script setup lang="ts">
// utils
function hourToTimeStr(val: number) {
    console.log("h2t", val)
    if (!val) { return "" }
    const h = `${~~val}`.padStart(2, "0")
    const min = `${~~((val - +h) * 60)}`.padStart(2, "0")
    return h + ":" + min
}
function timeStrToHour(val: string) {
    console.log("t2h", val)
    if (!val) { return 0 }
    const [h, min] = val.split(":")
    return +h + +min / 60
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
    d__hourly.value = d__base.value / d__day.value / 7.5
}

// 残業
const d__legalOvertime = ref(0) // 法定外
const d__fixedOvertime = ref(0) // 法定内
const d__overtime = ref(20) // 合計
const d__overtimePay = ref(0)
const fixedOvertime = computed({ get() { { return hourToTimeStr(d__fixedOvertime.value) } }, set(val) { d__fixedOvertime.value = timeStrToHour(val) } })
const overtime = computed({ get() { { return hourToTimeStr(d__overtime.value) } }, set(val) { d__overtime.value = timeStrToHour(val) } })
const overtimePay = computed(() => d__overtime.value * d__hourly.value * 1.25)

// 深夜
const d__midnight = ref(0)
const midnight = computed({ get() { { return hourToTimeStr(d__midnight.value) } }, set(val) { d__midnight.value = timeStrToHour(val) } })

// 不就労
const d__unemployed = ref(0)
const unemployed = computed({ get() { { return hourToTimeStr(d__unemployed.value) } }, set(val) { d__unemployed.value = timeStrToHour(val) } })

// 所得
const total = computed(() => d__base.value + ((d__overtime.value + d__midnight.value) * 1.25 + d__fixedOvertime.value - d__unemployed.value) * d__hourly.value)

// 給与
const taxIncluded = computed(() => total.value)

</script>

<template>
    <div class="wrapper">
        <div class="container">
            <div class="tables_1">
                <table>
                    <caption>契約内容</caption>
                    <tr>
                        <td>基本給</td>
                        <td><input type="number" id="base" v-model="base" step="1000">円</td>
                    </tr>
                    <tr>
                        <td>所定労働日数</td>
                        <td><input type="number" id="day" v-model="day">日</td>
                    </tr>
                    <tr>
                        <td>時給 <button @click="resetHourly">計算</button> </td>
                        <td> <input type="number" id="hourly" v-model="hourly" step="50">円/h</td>
                    </tr>
                </table>
                <table class="grid-2">
                    <caption>勤務実績</caption>
                    <tr>
                        <td>残業時間</td>
                        <td> <input type="time" id="overtime" v-model="overtime" step="600">h </td>
                    </tr>
                    <tr>
                        <td>（法定内残業時間</td>
                        <td> <input type="time" id="fixed-overtime" v-model="fixedOvertime" step="600">h ）</td>
                    </tr>
                    <tr>
                        <td>深夜時間</td>
                        <td> <input type="time" id="midnight" v-model="midnight" step="600">h </td>
                    </tr>
                    <tr>
                        <td>不就労</td>
                        <td> <input type="time" id="unemployed" v-model="unemployed">h </td>
                    </tr>
                </table>
            </div>
            <div>
                <table>
                    <caption>給与/所得</caption>
                    <tr>
                        <td>残業代</td>
                        <td> <input type="number" id="overtime_pay" v-model="overtimePay" readonly>円
                        </td>
                    </tr>
                    <tr>
                        <td>給与</td>
                        <td> <input type="number" id="tax_included" v-model="taxIncluded" readonly>円
                        </td>
                    </tr>
                </table>
            </div>
            <div>
                <table>
                    <caption>utility</caption>
                    <tr>
                        <td>時間</td>
                        <td><input type="time" id=""></td>
                    </tr>
                </table>
            </div>
        </div>
    </div>
</template>

<style>
.wrapper {
    display: flex;
    justify-content: center;
}

.container {
    max-width: 80%;
}

.container>div {
    margin-block: 20px;
}

.tables_1 {
    display: flex;
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
    display: block;
}

table,
*.tr,
*.td {
    margin: 0;
}
</style>