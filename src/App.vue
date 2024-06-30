<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import { useToast } from 'vue-toastification';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';
import { ref, computed, onMounted } from 'vue';

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));
  
  if(savedTransactions){
    transactions.value = savedTransactions;
  }
});

//Get Total Balance
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// Submit transaction
const handleTransactionSubmitted = (transactionData) => {
  const newId = generateUniqueId();
  transactions.value.push({
    id: newId,
    text: transactionData.text,
    amount: transactionData.amount,
  });
  saveTransactionToLS();
  console.log(newId);
  toast.success('Transaction added.');
};

const generateUniqueId = () => {
  const lastTransaction = transactions.value[transactions.value.length - 1];
  return lastTransaction ? lastTransaction.id + 1 : 1;
}

// Delete transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );
  saveTransactionToLS();
  toast.success('Transaction deleted.');
};

//Save to localstorage
const saveTransactionToLS = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
}
</script>

<template>
  <Header />
  <div class="container">
    <!-- adding "+" to show as number not string -->
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expenses="+expenses" />
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>
