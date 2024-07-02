<script>
import axios from 'axios'

export default {
    data() {
        return {
            intial_currency: "USD",
            intial_currency_nominal: "",
            required_currency_nominal: "",
            error: "",
            info: null,
            currencies: null,
            reverse: false,
        }
    },

    methods: {
        pageLoaded() {
            axios.get('https://www.cbr-xml-daily.ru/daily_json.js')
                .then(res => {
                    this.info = res.data
                    this.currencies = Object.values(this.info.Valute);
                })

        },

        reverseCurrency() {
            this.reverse = this.reverse == false ? true : false
            this.getCurrency()
        },

        getCurrency() {
            this.intial_currency_nominal = this.intial_currency_nominal.replace(',', '.')
            if (!this.intial_currency) {
                this.error = "Выберите валюту.";
                return;
            }
            if (isNaN(this.intial_currency_nominal)) {
                this.error = "Введите корректное число.";
                this.required_currency_nominal = ""
                return;
            }
            if (this.intial_currency_nominal.trim() == "") {
                this.required_currency_nominal = ""
                this.error = ""
                return;
            }
            
            this.error = ""

            const selectedCurrency = this.currencies.find(currency => currency.CharCode === this.intial_currency);
            
            if (!selectedCurrency) {
                this.error = "Валюта не найдена.";
                return;
            }

            var rubleValue

            if (!this.reverse) {
                rubleValue = parseFloat(this.intial_currency_nominal) / (selectedCurrency.Value / selectedCurrency.Nominal);
                this.required_currency_nominal = rubleValue.toFixed(4) + ' ' + this.intial_currency;
            }
            else {
                rubleValue = (selectedCurrency.Value / selectedCurrency.Nominal) * parseFloat(this.intial_currency_nominal);
                this.required_currency_nominal = rubleValue.toFixed(4) + ' RUB';
            }

            
        },

        resetInputs() {
            this.intial_currency = "USD"
            this.intial_currency_nominal = ""
            this.required_currency_nominal = ""
            this.error = ""
        }
    },

    mounted() {
        this.pageLoaded()
    }
}
</script>

<template>
    <div class="wrapper">
        <div class="header">
            <h1 v-if="this.reverse == false">Курс RUB {{ intial_currency != null ? "к " + intial_currency : "" }}</h1>
            <h1 v-else>Курс {{ intial_currency }} к RUB</h1>
            <button @click="reverseCurrency()"><svg width="21px" height="21px" viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" transform="translate(4 2)"><path d="m4.5 8.5-4 4 4 4"/><path d="m12.5 12.5h-12"/><path d="m8.5.5 4 4-4 4"/><path d="m12.5 4.5h-12"/></g></svg></button>
        </div>
        <p>Актуальная информация по ЦБ РФ</p>
        <div class="funds-count">
            <div>
                <input maxlength="20"placeholder="Количество" type="text" v-model="intial_currency_nominal" @change="getCurrency()">
                <select v-model="intial_currency" @change="this.intial_currency = $event.target.value, getCurrency()">
                    <option v-for="currency in currencies" :key="currency.ID" :value="currency.CharCode">
                        {{ currency.Name }} ({{ currency.CharCode }})
                    </option>
                </select>
            </div>
            <div>
                <input style="text-align: center;" disabled type="text" v-model="required_currency_nominal">
            </div>
        </div>
        <button @click="resetInputs()"><svg viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg" fill="#000000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <g fill="none" fill-rule="evenodd" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" transform="matrix(0 1 1 0 2.5 2.5)"> <path d="m3.98652376 1.07807068c-2.38377179 1.38514556-3.98652376 3.96636605-3.98652376 6.92192932 0 4.418278 3.581722 8 8 8s8-3.581722 8-8-3.581722-8-8-8"></path> <path d="m4 1v4h-4" transform="matrix(1 0 0 -1 0 6)"></path> </g> </g></svg></button>
        <p class="error">{{ error }}</p>
    </div>
</template>

<style scoped>
.wrapper {
    width: 900px;
    height: 500px;
    border-radius: 10px;
    background: #f2f2f2;
    box-shadow: 0px 0px 20px 0px #232323;
    padding: 15px;
    text-align: center;
    position: relative;
}

.wrapper h1 {
    margin-top: 50px;
}

.wrapper p {
    margin-top: 20px;
}

.funds-count {
    display: flex;
    align-items: center;
    flex-direction: column;
}

.funds-count div {
    /* margin: 10px; */
    display: flex;
    flex-direction: column;
    width: 80%;
}

.funds-count input, .funds-count select {
    border: gray 1px solid;
    padding: 10px;
    background: #ffffff;
    margin: 10px;
    outline: none;
    font-size: 16px;
}

button {
    margin: 0 auto;
    width: 50px;
    padding: 10px;
}

button svg {
    width: 30px;
    height: 30px;
    transition: all .1s linear;
}

.error {
    color: red;
    font-size: 1.5em;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%, -50%);
    margin: 0 !important;
}

button svg:hover {
    rotate: -180deg;
}

</style>
