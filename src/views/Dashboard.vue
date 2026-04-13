<template>
  <div class="dashboard p-4">
    <div class="wrapper-card grid lg:grid-cols-4 grid-cols-1 md:grid-cols-2 gap-2 mt-5">
      <div class="card bg-white dark:bg-gray-800 w-full rounded-md p-5 border dark:border-gray-700 flex">
        <div class="p-2 max-w-sm">
          <div class="bg-orange-200 rounded-full w-14 h-14 text-lg p-3 text-orange-600 mx-auto">
            <svg xmlns="http://www.w3.org/2000/svg" width="30px" height="30px" viewBox="0 0 24 24">
              <path fill="currentColor"
                d="M10 16V8a2 2 0 0 1 2-2h9V5c0-1.1-.9-2-2-2H5a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14c1.1 0 2-.9 2-2v-1h-9a2 2 0 0 1-2-2zm3-8c-.55 0-1 .45-1 1v6c0 .55.45 1 1 1h9V8h-9zm3 5.5c-.83 0-1.5-.67-1.5-1.5s.67-1.5 1.5-1.5s1.5.67 1.5 1.5s-.67 1.5-1.5 1.5z" />
            </svg>
          </div>
        </div>
        <div class="block p-2 w-full">
          <p class="font-semibold text-gray-900 dark:text-gray-200 text-xl">
            {{ formatCurrency(role === 'admin' ? summary.totalPayouts : summary.totalWithdraw) }}
          </p>
          <h2 class="font-normal text-gray-400 text-md mt-1">Total Penarikan</h2>
        </div>
      </div>

      <div class="card bg-white dark:bg-gray-800 w-full rounded-md p-5 border dark:border-gray-700 flex">
        <div class="p-2 max-w-sm">
          <div class="bg-green-200 rounded-full w-14 h-14 text-lg p-3 text-green-600 mx-auto">
            <svg xmlns="http://www.w3.org/2000/svg" width="30px" height="30px" viewBox="0 0 24 24">
              <path fill="currentColor"
                d="M5 22h14a2 2 0 0 0 2-2V9a1 1 0 0 0-1-1h-3v-.777c0-2.609-1.903-4.945-4.5-5.198A5.005 5.005 0 0 0 7 7v1H4a1 1 0 0 0-1 1v11a2 2 0 0 0 2 2zm12-12v2h-2v-2h2zM9 7c0-1.654 1.346-3 3-3s3 1.346 3 3v1H9V7zm-2 3h2v2H7v-2z" />
            </svg>
          </div>
        </div>
        <div class="block p-2 w-full">
          <p class="font-semibold text-gray-900 dark:text-gray-200 text-xl">
            {{ formatCurrency(role === 'admin' ? summary.totalDeposits : summary.totalDeposit) }}
          </p>
          <h2 class="font-normal text-gray-400 text-md mt-1">Total Setoran</h2>
        </div>
      </div>

      <div class="card bg-white dark:bg-gray-800 w-full rounded-md p-5 border dark:border-gray-700 flex">
        <div class="p-2 max-w-sm">
          <div class="bg-red-200 rounded-full w-14 h-14 text-lg p-3 text-red-600 mx-auto">
            <svg xmlns="http://www.w3.org/2000/svg" width="30px" height="30px" viewBox="0 0 32 32">
              <path fill="currentColor"
                d="M26.749 24.93A13.99 13.99 0 1 0 2 16a13.899 13.899 0 0 0 3.251 8.93l-.02.017c.07.084.15.156.222.239c.09.103.187.2.28.3c.28.304.568.596.87.87c.092.084.187.162.28.242c.32.276.649.538.99.782c.044.03.084.069.128.1v-.012a13.901 13.901 0 0 0 16 0v.012c.044-.031.083-.07.128-.1c.34-.245.67-.506.99-.782c.093-.08.188-.159.28-.242c.302-.275.59-.566.87-.87c.093-.1.189-.197.28-.3c.071-.083.152-.155.222-.24ZM16 8a4.5 4.5 0 1 1-4.5 4.5A4.5 4.5 0 0 1 16 8ZM8.007 24.93A4.996 4.996 0 0 1 13 20h6a4.996 4.996 0 0 1 4.993 4.93a11.94 11.94 0 0 1-15.986 0Z" />
            </svg>
          </div>
        </div>
        <div class="block p-2 w-full">
          <p class="font-semibold text-gray-900 dark:text-gray-200 text-xl">
            {{ role === 'admin' ? summary.totalCustomers : summary.transactionCount }}
          </p>
          <h2 class="font-normal text-gray-400 text-md mt-1">{{ role === 'admin' ? 'Total Pelanggan'
            : 'Jumlah Transaksi' }}</h2>
        </div>
      </div>

      <div class="card bg-white dark:bg-gray-800 w-full rounded-md p-5 border dark:border-gray-700 flex">
        <div class="p-2 max-w-sm">
          <div class="bg-purple-200 rounded-full w-14 h-14 text-lg p-3 text-purple-600 mx-auto">
            <svg xmlns="http://www.w3.org/2000/svg" width="30px" height="30px" viewBox="0 0 16 16">
              <path fill="currentColor" d="M15 13v1H1.5l-.5-.5V0h1v13h13Z" />
              <path fill="currentColor"
                d="M13 3.207L7.854 8.354h-.708L5.5 6.707l-3.646 3.647l-.708-.708l4-4h.708L7.5 7.293l5.146-5.147h.707l2 2l-.707.708L13 3.207Z" />
            </svg>
          </div>
        </div>
        <div class="block p-2 w-full">
          <p class="font-semibold text-gray-900 dark:text-gray-200 text-xl">
            {{ formatCurrency(role === 'admin' ? summary.totalBalance : summary.currentBalance) }}
          </p>
          <h2 class="font-normal text-gray-400 text-md mt-1">Saldo Saat Ini</h2>
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 mt-5">
      <div :class="role === 'admin' ? 'lg:col-span-2' : 'lg:col-span-3'"
        class="bg-white dark:bg-gray-800 p-5 w-full rounded-md border dark:border-gray-700">
        <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">Transaksi Terakhir</h2>
        <p class="text-gray-400 font-normal">Daftar aktivitas transaksi terbaru</p>
        <div class="wrapping-table mt-10 overflow-x-auto">
          <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
            <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
              <tr>
                <th class="px-6 py-3">Transaksi</th>
                <th class="px-6 py-3">Tanggal & Waktu</th>
                <th class="px-6 py-3">Jumlah</th>
                <th v-if="role === 'user'" class="px-6 py-3">Penalti</th>
                <th class="px-6 py-3">Status</th>
              </tr>
            </thead>
            <tbody>
              <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700" v-for="items in tableTransaction"
                :key="items.id">
                <td class="px-6 py-4">{{ translateTransaction(items.transaction || items.type) }}</td>
                <td class="px-6 py-4">{{ items.datetime || items.date }}</td>
                <td class="px-6 py-4 font-semibold text-gray-800 dark:text-gray-200">{{ typeof items.amount === 'number'
                  ? formatCurrency(items.amount) : items.amount || formatCurrency(items.displayAmount) }}</td>
                <td v-if="role === 'user'" class="px-6 py-4 text-red-500">{{ formatCurrency(items.penalty) }}</td>
                <td class="px-6 py-4">
                  <span class="px-3 py-1 rounded-md capitalize"
                    :class="getStatusClass(items.statusTransaction || items.status)">
                    {{ translateStatus(items.statusTransaction || items.status) }}
                  </span>
                </td>
              </tr>
              <tr v-if="tableTransaction.length === 0">
                <td :colspan="role === 'user' ? 5 : 4" class="px-6 py-4 text-center">Tidak ada transaksi terbaru</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-if="role === 'admin'"
        class="lg:col-span-1 bg-white dark:bg-gray-800 p-5 border dark:border-gray-700 rounded-md w-full">
        <div>
          <h2 class="text-lg font-semibold dark:text-gray-200">PENGGUNA BARU</h2>
          <p class="text-gray-400">Anggota yang baru saja bergabung.</p>
        </div>
        <div class="divide-y h-96 mt-5 dark:divide-gray-700 overflow-y-auto">
          <div class="p-3 w-full" v-for="user in recentUsers" :key="user.id">
            <div class="flex gap-5 items-center">
              <div class="bg-blue-100 text-blue-600 rounded-full w-10 h-10 flex items-center justify-center font-bold">
                {{ (user.full_name || user.username).charAt(0).toUpperCase() }}
              </div>
              <div>
                <h2 class="dark:text-gray-200 font-semibold">{{ user.full_name || user.username }}</h2>
                <p class="text-sm text-gray-400">@{{ user.username }}</p>
              </div>
            </div>
          </div>
          <div class="p-3 w-full text-center text-gray-400" v-if="recentUsers.length === 0">
            Tidak ada pengguna ditemukan
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const role = localStorage.getItem("role"); // 'admin' atau 'user'

const summary = ref({});
const tableTransaction = ref([]);
const recentUsers = ref([]);

const formatCurrency = (value) => {
  return new Intl.NumberFormat("id-ID", {
    style: "currency",
    currency: "IDR",
    minimumFractionDigits: 0,
  }).format(value || 0);
};

const translateTransaction = (text) => {
  if (!text) return "-";
  return text
    .toLowerCase()
    .replace("deposit from", "Setoran dari")
    .replace("withdrawal by", "Penarikan oleh")
    .replace("deposit", "Setoran")
    .replace("withdraw", "Penarikan");
};

const translateStatus = (status) => {
  const map = { completed: 'Selesai', progress: 'Proses', success: 'Berhasil', failed: 'Gagal' };
  return map[status] || status;
};

const getStatusClass = (status) => {
  if (['completed', 'success'].includes(status)) return 'text-green-800 bg-green-300';
  if (status === 'progress') return 'text-purple-800 bg-purple-300';
  return 'text-red-800 bg-red-300';
};

const fetchDashboardData = async () => {
  try {
    const token = localStorage.getItem("token");
    const endpoint = role === 'admin'
      ? "https://alentest.my.id/tabungan/api/dashboard"
      : "https://alentest.my.id/tabungan/api/user/dashboard";

    const response = await fetch(endpoint, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
    });

    if (!response.ok) throw new Error("Unauthorized");

    const result = await response.json();

    summary.value = result.summary;
    tableTransaction.value = role === 'admin' ? result.latestTransactions : result.recentTransactions;
    recentUsers.value = result.recentUsers || [];

  } catch (error) {
    router.push({ name: "Login" });
  }
};

onMounted(() => {
  fetchDashboardData();
});
</script>