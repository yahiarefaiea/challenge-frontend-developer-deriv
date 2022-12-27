<template>
  <div class="home">
    <h1>Deriv - Currency Converter</h1>
    <input type="number" v-model="from_value">
    <select v-model="base_currency">
      <option v-for="currency in payout_currencies">{{currency}}</option>
    </select>
    <table>
      <thead>
        <tr>
          <th>Currency</th>
          <th>Value</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(rate, currency) in exchange_rates?.rates || {}">
          <th>{{currency}}</th>
          <th>{{formatRate({rate: rate * from_value, currency})}}</th>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data: () => ({
    connectionUrl: 'wss://ws.binaryws.com/websockets/v3?app_id=1089',
    connection: null,
    from_value: 1,
    twoDecimalsCurrencies: ['USD', 'AUD', 'GBP'],
    payout_currencies: [],
    exchange_rates: {},
    base_currency: 'USD'
  }),
  watch: {
    base_currency() {
      this.updateRates()
    }
  },
  methods: {
    updateRates() {
      this.connection.send(JSON.stringify({payout_currencies: 1}))
      this.connection.send(JSON.stringify({
        exchange_rates: 1,
        base_currency: this.base_currency
      }))
    },
    parseRates(message) {
      const data = JSON.parse(message.data)
      
      if(data.msg_type === 'payout_currencies')
        this.payout_currencies = data.payout_currencies
      else if(data.msg_type === 'exchange_rates')
        this.exchange_rates = data.exchange_rates
    },
    formatRate({rate, currency}) {
      if(this.twoDecimalsCurrencies.includes(currency))
        return Math.round(rate * 100) / 100
      return rate
    }
  },
  created() {
    this.connection = new WebSocket(this.connectionUrl)
    this.connection.onopen = this.updateRates
    this.connection.onmessage = this.parseRates
  }
}
</script>
