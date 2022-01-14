<template>
  <div class="content">
    <div
      v-show="!details"
      class="month-picker-container"
    >
      <MonthPickerInput
        v-model="date"
        :defaultYear="2020"
        placeholder="Select a date range"
        :maxDate="maxDate"
        :minDate="minDate"
        clearable
        range
      />
      <button
        @click="filterByDate"
        class="filter-button"
      >
        Filter date
      </button>
      <button
        @click="loadTransactions"
        class="filter-button"
      >
        Reload all
      </button>
    </div>
    <div
      v-if="!details"
      class="transactions-table"
    >
      <div
        v-show="loading"
        class="loader"
      >
        loading...
      </div>
      <div
        v-show="!loading && isEmpty"
        class="emptyState"
      >
        No transactions were found.
      </div>
      <table v-show="!loading && !isEmpty">
        <tr class="transactions-header">
          <th>Account</th>
          <th>Date</th>
          <th>Currency</th>
          <th>Amount</th>
        </tr>
        <tr
          v-for="transaction in transactions"
          :key="transaction.prismaid"
          class="transaction"
          @click="openDetails(transaction)"
        >
          <td>{{transaction.account}}</td>
          <td>{{transaction.transactionDate}}</td>
          <td>{{transaction.currency}}</td>
          <td>{{transaction.amount}}</td>
        </tr>
      </table>
    </div>
    <div
      v-else
      class="details"
    >
      <div class="details-card">
        <div class="header-container">
          <span @click="closeDetails" class="back-icon">X</span>
          <h2>Transaction Details</h2>
        </div>
        <div class="transaction-details-container">
          <div class="transaction-details">
            <h3>ID: </h3>
            <span>{{transactionDetails.id}}</span>
          </div>
          <div class="transaction-details">
            <h3>Account: </h3>
            <span>{{transactionDetails.account}}</span>
          </div>
          <div class="transaction-details">
            <h3>Amount: </h3>
            <span>{{transactionDetails.amount}}</span>
          </div>
          <div class="transaction-details">
            <h3>Category: </h3>
            <span>{{transactionDetails.category}}</span>
          </div>
          <div class="transaction-details">
            <h3>Created At: </h3>
            <span>{{transactionDetails.createdAt}}</span>
          </div>
          <div class="transaction-details">
            <h3>Currency: </h3>
            <span>{{transactionDetails.currency}}</span>
          </div>
          <div class="transaction-details">
            <h3>Description: </h3>
            <span>{{transactionDetails.description}}</span>
          </div>
          <div class="transaction-details">
            <h3>Reference: </h3>
            <span>{{transactionDetails.reference}}</span>
          </div>
          <div class="transaction-details">
            <h3>Status: </h3>
            <span>{{transactionDetails.status}}</span>
          </div>
          <div class="transaction-details">
            <h3>Date: </h3>
            <span>{{transactionDetails.transactionDate}}</span>
          </div>
          <div class="transaction-details">
            <h3>Updated At: </h3>
            <span>{{transactionDetails.updatedAt}}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { MonthPickerInput } from 'vue-month-picker'
import axios from "axios"

export default {
  name: 'Home',
  components: {
    MonthPickerInput,
  },
  data() {
    return {
      transactions: [],
      transactionDetails: {},
      details: false,
      loading: false,
      date: {},
    }
  },
  async created() {
    await this.loadTransactions()
  },
  computed: {
    startDate() {
      const months = {
        January: "01",
        February: "02",
        March: "03",
        April: "04",
        May: "05",
        June: "06",
        July: "07",
        August: "08",
        September: "09",
        November: "11",
        December: "12",
      }
      return months[this.date.rangeFromMonth]
    },
    endDate() {
      const months = {
        January: "01",
        February: "02",
        March: "03",
        April: "04",
        May: "05",
        June: "06",
        July: "07",
        August: "08",
        September: "09",
        November: "11",
        December: "12",
      }
      return months[this.date.rangeToMonth]
    },
    maxDate() {
      return new Date("2021-12-31");
    },
    minDate() {
      return new Date("2020-01-01");
    },
    isEmpty() {
      return this.transactions === null ||this.transactions.length === 0
    }
  },
  methods: {
    openDetails(transaction) {
      this.transactionDetails = transaction
      this.details = true
    },
    closeDetails() {
      this.details = false
    },
    showDate (date) {
			this.date = date
		},
    async filterByDate() {
      this.loading = true
      try {
        const transactions = await axios.post("http://localhost:4000/api", {
          query:
            `{transactionsInPeriod(start: "${this.date.year}-${this.startDate}", end: "${this.date.year}-${this.endDate}" ) {prismaid, id, createdAt, updatedAt, transactionDate, account, description, category, reference, currency, amount, status}}`,
        });
        this.transactions = transactions.data.data.transactionsInPeriod;
      } catch (err) {
        console.log(err);
      } finally {
        this.loading = false
      }
    },
    async loadTransactions() {
    this.loading = true
    try {
      const transactions = await axios.post("http://localhost:4000/api", {
        query:
          "{transactions {prismaid, id, createdAt, updatedAt, transactionDate, account, description, category, reference, currency, amount, status}}",
      });
      this.transactions = transactions.data.data.transactions;
    } catch (err) {
      console.log(err);
    } finally {
      this.loading = false
    }
    }
  }
}
</script>

<style scoped>
.content {
  width: 100%;
  position: relative;
}

.transactions-table {
  max-height: 800px;
  overflow-y: scroll;
  border: 2px solid #f2f2f2;
  border-radius: 8px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

.transactions-header {
  background-color: #f2f2f2;
}

.transaction:hover {
  cursor: pointer;
  background-color: lightgrey;
}

th {
  text-align: left;
  font-size: 18px;
  padding: 18px 8px;
}
td {
  text-align: left;
  padding: 12px 8px 12px 8px;
}

.details-card {
  width: 100%;
  border: 2px solid #f2f2f2;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  margin-top: 90px;
}

.header-container {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 24px;
  padding: 18px;
}

.header-container h2 {
  flex: 1;
}

.transaction-details-container {
  width: 100%;
  padding: 12px;
  display: flex;
  max-height: 400px;
  flex-wrap: wrap;
  justify-content: space-between;
  flex-direction: column;
}

.transaction-details {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
  align-items: flex-start;
  margin-right: 32px;
  border-radius: 8px;
  background-color: lightyellow;
  padding: 12px;
}

.transaction-details h3 {
  margin: 0px;
}
.back-icon {
  cursor: pointer;
  font-size: 24px;

}

span {
  margin-top: 12px;
}

.month-picker-container {
  display: flex;
  width: 100%;
  margin-bottom: 12px;
}


.filter-button {
  margin-left: 26px;
  cursor: pointer;
  width: 100px;
  border: 2px solid #f2f2f2;
  border-radius: 8px;
}

.filter-button:hover {
  background-color: lightgray;
}
.filter-button:last-child {
  margin-left: 8px;
}
</style>
