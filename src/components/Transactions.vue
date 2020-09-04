<template>
  <div>
    <h1>
      Transaction Feed <span class="account">Account: {{ accountNumber }}</span>
    </h1>
    <h2 class="balance">Starting Balance: ${{ startingBalance.toFixed(2) }}</h2>
    <h3 class="transaction-list-header">Pending Transactions</h3>
    <div
      v-for="transaction in this.pending"
      :key="transaction.transaction.id"
      class="transaction"
    >
      <div class="merchant">
        {{ transaction.transaction.MerchantName }}
        <span class="merchant-id"
          >({{ transaction.transaction.MerchantId }})</span
        >
      </div>
      <div
        :class="transaction.transaction.TransactionTypeId.toLowerCase()"
        class="amount"
      >
        ${{ transaction.transaction.Amount.toFixed(2) }}
      </div>
      <div class="description">{{ transaction.transaction.Description }}</div>
      <div class="date">
        {{ new Date(transaction.transaction.TransactionDate).getMonth() }}/{{
          new Date(transaction.transaction.TransactionDate).getDate()
        }}/{{
          new Date(transaction.transaction.TransactionDate).getFullYear()
        }}
        {{ new Date(transaction.transaction.TransactionDate).getHours() }}:{{
          new Date(transaction.transaction.TransactionDate).getMinutes()
        }}
      </div>
      <div class="current-balance">${{ transaction.balance.toFixed(2) }}</div>
    </div>
    <h3 class="transaction-list-header">Completed Transactions</h3>
    <div
      v-for="transaction in this.completed"
      :key="transaction.transaction.id"
      class="transaction"
    >
      <div class="merchant">
        {{ transaction.transaction.MerchantName }}
        <span class="merchant-id"
          >({{ transaction.transaction.MerchantId }})</span
        >
      </div>
      <div
        :class="transaction.transaction.TransactionTypeId.toLowerCase()"
        class="amount"
      >
        ${{ transaction.transaction.Amount.toFixed(2) }}
      </div>
      <div class="description">{{ transaction.transaction.Description }}</div>
      <div class="date">
        {{ new Date(transaction.transaction.TransactionDate).getMonth() }}/{{
          new Date(transaction.transaction.TransactionDate).getDate()
        }}/{{
          new Date(transaction.transaction.TransactionDate).getFullYear()
        }}
        {{ new Date(transaction.transaction.TransactionDate).getHours() }}:{{
          new Date(transaction.transaction.TransactionDate).getMinutes()
        }}
      </div>
      <div class="current-balance">${{ transaction.balance.toFixed(2) }}</div>
    </div>
    <h2 class="balance">Ending Balance: ${{ endingBalance.toFixed(2) }}</h2>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  name: "Transactions",
  data() {
    return {
      startingBalance: null,
      currentBalance: null,
      endingBalance: null,
      accountNumber: null,
      pending: [],
      completed: [],
    };
  },
  computed: mapState({
    statement: (state) => state.statement,
  }),
  created() {
    this.accountNumber = this.statement.Statement.Transactions[0].AccountNumber;
    //Get starting balance from first transaction
    this.currentBalance = this.endingBalance = this.startingBalance = this.statement.Statement.Transactions[0].AvailableBalance;
    //Parse transactions
    let transactions = this.statement.Statement.Transactions.reverse();
    //Parse Pending
    transactions.forEach((transaction) => {
      if (!transaction.Billed) {
        //Increment Ending Balance
        if (transaction.TransactionTypeId === "Debit") {
          this.currentBalance -= transaction.Amount;
        } else if (transaction.TransactionTypeId === "Credit") {
          this.currentBalance += transaction.Amount;
          this.endingBalance += transaction.Amount;
        }
        //Push to pending array
        this.pending.push({
          transaction: transaction,
          balance: this.currentBalance,
        });
      }
    });
    //Parse completed
    transactions.forEach((transaction) => {
      if (transaction.Billed) {
        //Increment Ending Balance
        if (transaction.TransactionTypeId === "Debit") {
          this.currentBalance -= transaction.Amount;
          this.endingBalance -= transaction.Amount;
        } else if (transaction.TransactionTypeId === "Credit") {
          this.currentBalance += transaction.Amount;
          this.endingBalance += transaction.Amount;
        }
        //Push to completed array
        this.completed.push({
          transaction: transaction,
          balance: this.currentBalance,
        });
      }
    });
  },
};
</script>

<style>
.account {
  font-size: 18px;
  font-weight: normal;
}
.transaction-list-header {
  background-color: #ccc;
  padding: 5px;
}
.balance {
  padding: 15px 0;
}
.transaction {
  border-bottom: 1px #333 solid;
  padding: 15px 0;
  font-size: 16px;
}
.merchant {
  display: inline-block;
  padding-right: 5px;
  font-size: 18px;
  font-weight: bold;
  width: 200px;
}
.merchant-id {
  color: #666;
  font-size: 12px;
  font-weight: unset;
  display: none;
  padding-right: 5px;
}
.amount {
  display: inline-block;
  font-size: 18px;
  font-weight: bold;
  float: right;
}
.current-balance {
  display: inline-block;
  font-size: 18px;
  color: #666;
  float: right;
}
.description {
  display: block;
}
.date {
  display: inline-block;
  padding-right: 5px; 
  width: 200px;

}
.debit {
  color: red;
}
.credit {
  color: green;
}
</style>
