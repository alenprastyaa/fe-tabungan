<template>
    <div>
        <div class="print:hidden">
            <div class="p-8">
                <div class="flex justify-between items-center mb-6">
                    <h1 class="text-2xl font-bold dark:text-white">Manajemen Tabungan</h1>
                </div>

                <div v-if="globalMessage" :class="messageClass"
                    class="mb-6 p-4 text-sm rounded-md flex justify-between items-center">
                    <span>{{ globalMessage }}</span>
                    <button @click="globalMessage = ''" class="font-bold text-lg">&times;</button>
                </div>

                <div
                    class="bg-white dark:bg-gray-800 p-4 rounded-lg shadow mb-6 border dark:border-gray-700 flex flex-wrap items-end gap-4">
                    <div class="flex-1 min-w-[200px]">
                        <label class="block text-xs font-medium text-gray-500 dark:text-gray-400 mb-1">Tanggal
                            Lahir</label>
                        <input v-model="searchDob" type="date"
                            class="w-full px-3 py-2 text-sm border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-blue-500 transition-colors" />
                    </div>
                    <div class="flex gap-2">
                        <button @click="resetSearch" v-if="searchQuery || searchDob"
                            class="bg-gray-200 hover:bg-gray-300 text-gray-800 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white px-4 py-2 rounded-md text-sm font-medium transition-colors">
                            Reset Pencarian
                        </button>
                    </div>
                </div>

                <div v-if="isLoading" class="text-center dark:text-gray-300 py-10">
                    <span class="animate-pulse">Memuat data tabungan...</span>
                </div>

                <div v-else
                    class="bg-white dark:bg-gray-800 rounded-lg shadow border dark:border-gray-700 overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                            <thead class="bg-gray-50 dark:bg-gray-700">
                                <tr>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        ID</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Username</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Nama Lengkap</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Tanggal Lahir</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Alamat</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Total Saldo</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-center text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                        Aksi Transaksi</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-gray-200 dark:divide-gray-700">
                                <tr v-for="(item, index) in balances" :key="item.user_id"
                                    class="hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors">
                                    <td
                                        class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 dark:text-white">
                                        {{ index + 1 }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">{{
                                        item.username || '-' }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">{{
                                        item.full_name || '-' }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">{{
                                        formatTanggalIndo(item.dob) }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">{{
                                        item.address || '-' }}</td>
                                    <td
                                        class="px-6 py-4 whitespace-nowrap text-sm font-bold text-green-600 dark:text-green-400">
                                        {{ formatRupiah(item.balance || 0) }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-center space-x-2">
                                        <button @click="openDepositModal(item)"
                                            class="text-white bg-green-600 hover:bg-green-700 font-medium px-3 py-1 rounded-md transition-colors">Setor</button>
                                        <button @click="openWithdrawModal(item)"
                                            class="text-white bg-orange-600 hover:bg-orange-700 font-medium px-3 py-1 rounded-md transition-colors">Tarik</button>
                                        <button @click="openHistoryModal(item)"
                                            class="text-white bg-blue-600 hover:bg-blue-700 font-medium px-3 py-1 rounded-md transition-colors">Detail</button>
                                    </td>
                                </tr>
                                <tr v-if="balances.length === 0">
                                    <td colspan="7" class="px-6 py-10 text-center text-gray-500 dark:text-gray-400">
                                        Tidak ada data tabungan yang ditemukan.</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div
                        class="px-6 py-4 border-t dark:border-gray-700 flex justify-between items-center bg-gray-50 dark:bg-gray-800">
                        <button @click="prevPage" :disabled="currentPage === 1"
                            class="px-4 py-2 border rounded-md text-sm disabled:opacity-50 dark:text-white dark:border-gray-600">Sebelumnya</button>
                        <span class="text-sm dark:text-white">Halaman {{ currentPage }}</span>
                        <button @click="nextPage" :disabled="balances.length < currentLimit || balances.length === 0"
                            class="px-4 py-2 border rounded-md text-sm disabled:opacity-50 dark:text-white dark:border-gray-600">Selanjutnya</button>
                    </div>
                </div>
            </div>

            <div v-if="showDepositModal"
                class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm px-4">
                <div class="bg-white dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-md overflow-hidden relative">
                    <div class="px-6 py-4 border-b dark:border-gray-700 flex justify-between items-center">
                        <h2 class="text-xl font-bold dark:text-white">Setor Tabungan</h2>
                        <button @click="closeModal"
                            class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 text-2xl font-bold">&times;</button>
                    </div>
                    <div class="p-6">
                        <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">
                            User: <strong class="text-gray-900 dark:text-white">{{ selectedUser?.full_name ||
                                selectedUser?.username }}</strong>
                        </p>
                        <form @submit.prevent="submitDeposit" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Nominal
                                    Setoran</label>
                                <div class="relative">
                                    <span
                                        class="absolute left-3 top-1/2 -translate-y-1/2 text-sm font-medium text-gray-500 dark:text-gray-400">Rp</span>
                                    <input :value="depositAmountFormatted" @input="onDepositInput"
                                        @keypress="onlyNumberKey" type="text" inputmode="numeric" placeholder="0"
                                        required
                                        class="w-full pl-10 pr-3 py-2 border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-green-500 text-right tabular-nums" />
                                </div>
                                <p v-if="depositAmount" class="mt-1 text-xs text-gray-500 dark:text-gray-400">
                                    {{ formatRupiah(depositAmount) }}
                                </p>
                            </div>
                            <div class="flex justify-end pt-4 space-x-3 border-t dark:border-gray-700 mt-6">
                                <button type="button" @click="closeModal"
                                    class="px-4 py-2 text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-md transition-colors">Batal</button>
                                <button type="submit" :disabled="isProcessing || !depositAmount"
                                    class="bg-green-600 hover:bg-green-700 text-white px-6 py-2 rounded-md disabled:opacity-50">
                                    {{ isProcessing ? 'Memproses...' : 'Simpan Setoran' }}
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>

            <div v-if="showWithdrawModal"
                class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm px-4">
                <div class="bg-white dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-md overflow-hidden relative">
                    <div class="px-6 py-4 border-b dark:border-gray-700 flex justify-between items-center">
                        <h2 class="text-xl font-bold dark:text-white">Tarik Tabungan</h2>
                        <button @click="closeModal"
                            class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 text-2xl font-bold">&times;</button>
                    </div>
                    <div class="p-6">
                        <div class="mb-4 text-sm text-gray-600 dark:text-gray-400">
                            <p>User: <strong class="text-gray-900 dark:text-white">{{ selectedUser?.full_name ||
                                selectedUser?.username }}</strong></p>
                            <p>Saldo Saat Ini: <strong class="text-green-600 dark:text-green-400">{{
                                formatRupiah(selectedUser?.balance || 0) }}</strong></p>
                        </div>
                        <form @submit.prevent="submitWithdraw" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Nominal
                                    Penarikan</label>
                                <div class="relative">
                                    <span
                                        class="absolute left-3 top-1/2 -translate-y-1/2 text-sm font-medium text-gray-500 dark:text-gray-400">Rp</span>
                                    <input :value="withdrawAmountFormatted" @input="onWithdrawInput"
                                        @keypress="onlyNumberKey" type="text" inputmode="numeric" placeholder="0"
                                        required
                                        class="w-full pl-10 pr-3 py-2 border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-orange-500 text-right tabular-nums" />
                                </div>
                            </div>
                            <div>
                                <label
                                    class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Persentase
                                    Penalti (%)</label>
                                <input v-model.number="penaltyPercent" type="number" min="0" max="100" required
                                    class="w-full px-3 py-2 border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-orange-500" />

                                <div v-if="withdrawAmount"
                                    class="mt-3 p-3 bg-gray-50 dark:bg-gray-700 rounded-md border border-gray-200 dark:border-gray-600">
                                    <div class="flex justify-between text-sm mb-1">
                                        <span class="text-gray-600 dark:text-gray-400">Potongan Penalti ({{
                                            penaltyPercent }}%):</span>
                                        <span class="text-red-600 dark:text-red-400 font-medium">-{{
                                            formatRupiah(Math.floor(withdrawAmount * penaltyPercent / 100)) }}</span>
                                    </div>
                                    <div
                                        class="flex justify-between text-sm font-bold border-t border-gray-200 dark:border-gray-600 pt-2 mt-2">
                                        <span class="text-gray-800 dark:text-gray-200">Uang Diterima:</span>
                                        <span class="text-green-600 dark:text-green-400">{{ formatRupiah(withdrawAmount
                                            - Math.floor(withdrawAmount * penaltyPercent / 100)) }}</span>
                                    </div>
                                </div>
                            </div>
                            <div class="flex justify-end pt-4 space-x-3 border-t dark:border-gray-700 mt-6">
                                <button type="button" @click="closeModal"
                                    class="px-4 py-2 text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-md transition-colors">Batal</button>
                                <button type="submit" :disabled="isProcessing || !withdrawAmount"
                                    class="bg-orange-600 hover:bg-orange-700 text-white px-6 py-2 rounded-md disabled:opacity-50">
                                    {{ isProcessing ? 'Memproses...' : 'Tarik Dana' }}
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>

            <div v-if="showHistoryModal"
                class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm px-4">
                <div
                    class="bg-white dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-5xl overflow-hidden relative flex flex-col max-h-[90vh]">
                    <div class="px-6 py-4 border-b dark:border-gray-700 flex justify-between items-center shrink-0">
                        <h2 class="text-xl font-bold dark:text-white">Riwayat Transaksi: {{ selectedUser?.full_name ||
                            selectedUser?.username }}</h2>
                        <div class="flex items-center space-x-4">
                            <button @click="printHistoryAction"
                                class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-1.5 rounded-md text-sm font-medium transition-colors flex items-center gap-2">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24"
                                    stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4a2 2 0 00-2-2H9a2 2 0 00-2 2v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z" />
                                </svg>
                                Cetak Semua Riwayat
                            </button>
                            <button @click="closeModal"
                                class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 text-2xl font-bold">&times;</button>
                        </div>
                    </div>

                    <div
                        class="p-6 shrink-0 bg-gray-50 dark:bg-gray-800 border-b dark:border-gray-700 flex gap-4 items-end">
                        <div class="flex-1">
                            <label class="block text-xs font-medium text-gray-500 dark:text-gray-400 mb-1">Tipe
                                Transaksi</label>
                            <select v-model="historyFilters.type" @change="resetAndFetchHistory"
                                class="w-full px-3 py-2 text-sm border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-blue-500">
                                <option value="">Semua</option>
                                <option value="deposit">Setor (Deposit)</option>
                                <option value="withdraw">Tarik (Withdraw)</option>
                            </select>
                        </div>
                        <div class="flex-1">
                            <label class="block text-xs font-medium text-gray-500 dark:text-gray-400 mb-1">Mulai
                                Tanggal</label>
                            <input v-model="historyFilters.start_date" type="date" @change="resetAndFetchHistory"
                                class="w-full px-3 py-2 text-sm border border-gray-300 rounded-md dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-1 focus:ring-blue-500" />
                        </div>
                        <div>
                            <button @click="clearHistoryFilters" v-if="historyFilters.type || historyFilters.start_date"
                                class="bg-gray-200 hover:bg-gray-300 text-gray-800 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white px-4 py-2 rounded-md text-sm font-medium transition-colors">
                                Reset
                            </button>
                        </div>
                    </div>

                    <div class="p-6 overflow-y-auto flex-1">
                        <div v-if="historyLoading" class="text-center dark:text-gray-300 py-10">
                            <span class="animate-pulse">Memuat riwayat transaksi...</span>
                        </div>
                        <div v-else>
                            <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                                <thead class="bg-gray-50 dark:bg-gray-700">
                                    <tr>
                                        <th scope="col"
                                            class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Tanggal</th>
                                        <th scope="col"
                                            class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Tipe</th>
                                        <th scope="col"
                                            class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Nominal</th>
                                        <th scope="col"
                                            class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Penalti</th>
                                        <th scope="col"
                                            class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Total Akhir</th>
                                        <th scope="col"
                                            class="px-4 py-3 text-center text-xs font-medium text-gray-500 dark:text-gray-300 uppercase">
                                            Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y divide-gray-200 dark:divide-gray-700">
                                    <tr v-for="history in historyData" :key="history.id"
                                        class="hover:bg-gray-50 dark:hover:bg-gray-700">
                                        <td
                                            class="px-4 py-3 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">
                                            {{ formatDateTimeIndo(history.created_at) }}</td>
                                        <td class="px-4 py-3 whitespace-nowrap text-sm">
                                            <span
                                                :class="history.type === 'deposit' ? 'text-green-600 bg-green-100 dark:bg-green-900/30 px-2 py-1 rounded-md' : 'text-orange-600 bg-orange-100 dark:bg-orange-900/30 px-2 py-1 rounded-md'">
                                                {{ history.type === 'deposit' ? 'Setor' : 'Tarik' }}
                                            </span>
                                        </td>
                                        <td
                                            class="px-4 py-3 whitespace-nowrap text-sm font-medium text-gray-900 dark:text-white">
                                            {{ formatRupiah(history.amount) }}</td>
                                        <td class="px-4 py-3 whitespace-nowrap text-sm text-red-500">{{
                                            history.penalty_amount > 0 ? '-' + formatRupiah(history.penalty_amount) :
                                                '-' }}</td>
                                        <td
                                            class="px-4 py-3 whitespace-nowrap text-sm font-bold text-gray-900 dark:text-white">
                                            {{ formatRupiah(history.final_amount) }}</td>
                                        <td class="px-4 py-3 whitespace-nowrap text-sm text-center">
                                            <button @click="reprintTransaction(history)"
                                                class="text-white bg-indigo-500 hover:bg-indigo-600 px-3 py-1.5 rounded-md text-xs font-medium transition-colors">
                                                Cetak Ulang
                                            </button>
                                        </td>
                                    </tr>
                                    <tr v-if="historyData.length === 0">
                                        <td colspan="6" class="px-4 py-10 text-center text-gray-500 dark:text-gray-400">
                                            Tidak ada riwayat transaksi ditemukan.</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <div
                        class="px-6 py-4 border-t dark:border-gray-700 flex justify-between items-center bg-gray-50 dark:bg-gray-800 shrink-0">
                        <button @click="prevHistoryPage" :disabled="!historyMeta.has_prev"
                            class="px-4 py-2 border rounded-md text-sm disabled:opacity-50 dark:text-white dark:border-gray-600">Sebelumnya</button>
                        <span class="text-sm dark:text-white">Halaman {{ historyMeta.page }} dari {{
                            historyMeta.total_pages }} (Total: {{ historyMeta.total }})</span>
                        <button @click="nextHistoryPage" :disabled="!historyMeta.has_next"
                            class="px-4 py-2 border rounded-md text-sm disabled:opacity-50 dark:text-white dark:border-gray-600">Selanjutnya</button>
                    </div>
                </div>
            </div>

            <div v-if="showReceiptModal"
                class="fixed inset-0 z-[60] flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm px-4">
                <div
                    class="bg-white dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-sm overflow-hidden text-center p-6">
                    <div
                        class="w-16 h-16 bg-green-100 dark:bg-green-900 text-green-600 rounded-full flex items-center justify-center mx-auto mb-4">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                        </svg>
                    </div>
                    <h2 class="text-xl font-bold dark:text-white mb-2">Transaksi Berhasil!</h2>

                    <div v-if="receiptData"
                        class="bg-gray-50 dark:bg-gray-700 p-4 rounded-md mb-6 text-left w-full text-sm border dark:border-gray-600">
                        <div class="flex justify-between mb-2">
                            <span class="text-gray-600 dark:text-gray-400">Jenis Transaksi:</span>
                            <span class="font-semibold dark:text-white">
                                {{ receiptData.type === 'deposit' ? 'Setor Tunai'
                                    : 'Tarik Tunai' }}</span>
                        </div>
                        <div class="flex justify-between mb-2">
                            <span class="text-gray-600 dark:text-gray-400">Nominal:</span>
                            <span
                                :class="receiptData.type === 'deposit' ? 'text-green-600 dark:text-green-400' : 'text-orange-600 dark:text-orange-400'"
                                class="font-semibold">
                                {{ receiptData.type === 'deposit' ? '+' : '-' }} {{ formatRupiah(receiptData.amount ||
                                    0) }}
                            </span>
                        </div>
                        <div class="flex justify-between pt-2 border-t border-gray-200 dark:border-gray-600">
                            <span class="text-gray-600 dark:text-gray-400">Total Saldo:</span>
                            <span class="font-bold text-lg dark:text-white">{{ formatRupiah(receiptData.finalBalance ||
                                0) }}</span>
                        </div>
                    </div>

                    <div class="flex gap-3 justify-center">
                        <button @click="closeModal"
                            class="px-4 py-2 text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-md transition-colors font-medium">Tutup</button>
                        <button @click="printReceiptAction"
                            class="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2 rounded-md font-medium flex items-center gap-2">
                            Cetak Struk
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div id="print-area" class="hidden print:block bg-white text-black font-sans w-full mx-auto">
            <div v-if="printMode === 'receipt' && receiptData"
                class="w-full mx-auto border border-gray-300 p-4 rounded">
                <div class="text-center mb-3 border-b pb-3">
                    <h2 class="text-lg font-bold uppercase">{{ receiptData.type === 'deposit' ? 'Bukti Setoran'
                        : 'Bukti Penarikan' }}</h2>
                    <p class="text-xs text-gray-500 mt-1">Manajemen Tabungan App</p>
                </div>

                <div class="space-y-1 text-xs mb-4">
                    <div class="flex justify-between">
                        <span class="text-gray-600">Tanggal:</span>
                        <span class="font-medium">{{ formatDateTimeIndo(receiptData.date) }}</span>
                    </div>
                    <div class="flex justify-between">
                        <span class="text-gray-600">Nama:</span>
                        <span class="font-medium">{{ receiptData.user }}</span>
                    </div>
                    <div class="flex justify-between">
                        <span class="text-gray-600">Tipe:</span>
                        <span class="font-medium">{{ receiptData.type === 'deposit' ? 'Setor Tunai' : 'Tarik Tunai'
                        }}</span>
                    </div>
                </div>

                <div class="border-t border-b border-black py-2 mb-4 space-y-1">
                    <div class="flex justify-between font-bold text-sm">
                        <span>{{ receiptData.type === 'deposit' ? 'Nominal Setor:' : 'Nominal Tarik:' }}</span>
                        <span>{{ formatRupiah(receiptData.amount) }}</span>
                    </div>

                    <div v-if="receiptData.type === 'withdraw' && receiptData.penaltyAmount > 0"
                        class="flex justify-between text-xs text-gray-600">
                        <span>Potongan Penalti ({{ receiptData.penaltyPercent }}%):</span>
                        <span>-{{ formatRupiah(receiptData.penaltyAmount) }}</span>
                    </div>

                    <div v-if="receiptData.type === 'withdraw'"
                        class="flex justify-between font-bold text-sm mt-1 border-t border-dashed border-gray-400 pt-1">
                        <span>Uang Diterima:</span>
                        <span>{{ formatRupiah(receiptData.receivedAmount) }}</span>
                    </div>

                    <div class="flex justify-between font-bold text-sm mt-2 border-t border-black pt-2">
                        <span>Sisa Saldo:</span>
                        <span>{{ formatRupiah(receiptData.finalBalance) }}</span>
                    </div>
                </div>

                <div class="text-center text-[10px] text-gray-500">
                    <p>Terima kasih telah menggunakan layanan kami.</p>
                    <p class="mt-1">Struk ini adalah bukti transaksi yang sah.</p>
                </div>
            </div>

            <div v-if="printMode === 'history' && selectedUser">
                <div class="text-center mb-4 border-b border-black pb-3">
                    <h2 class="text-lg font-bold uppercase mb-1">Riwayat Transaksi</h2>
                    <p class="text-sm">Nama Nasabah: <strong>{{ selectedUser.full_name || selectedUser.username
                    }}</strong></p>
                    <p class="text-sm">Total Saldo Terakhir: <strong>{{ formatRupiah(selectedUser.balance || 0)
                    }}</strong></p>
                    <p class="text-[10px] text-gray-600 mt-1">Dicetak pada: {{ formatDateTimeIndo(new
                        Date().toISOString()) }}</p>
                </div>

                <table class="w-full text-left border-collapse border border-gray-400 text-xs">
                    <thead class="bg-gray-100">
                        <tr>
                            <th class="border border-gray-400 px-2 py-1 font-bold">Tgl & Waktu</th>
                            <th class="border border-gray-400 px-2 py-1 font-bold">Tipe</th>
                            <th class="border border-gray-400 px-2 py-1 font-bold text-right">Nominal</th>
                            <th class="border border-gray-400 px-2 py-1 font-bold text-right">Total Akhir</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="history in historyData" :key="history.id">
                            <td class="border border-gray-400 px-2 py-1 text-[10px]">{{
                                formatDateTimeIndo(history.created_at) }}</td>
                            <td class="border border-gray-400 px-2 py-1 text-[10px]">{{ history.type === 'deposit' ?
                                'Setor' : 'Tarik' }}</td>
                            <td class="border border-gray-400 px-2 py-1 text-right text-[10px]">{{
                                formatRupiah(history.amount) }}</td>
                            <td class="border border-gray-400 px-2 py-1 text-right font-bold text-[10px]">{{
                                formatRupiah(history.final_amount) }}</td>
                        </tr>
                        <tr v-if="historyData.length === 0">
                            <td colspan="4" class="border border-gray-400 px-2 py-4 text-center italic text-gray-500">
                                Tidak ada data transaksi.</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue';
import { useRouter } from 'vue-router';

const token = localStorage.getItem('token');
const baseUrl = 'https://alentest.my.id/tabungan';
const router = useRouter()

const balances = ref([]);
const isLoading = ref(true);
const globalMessage = ref('');
const messageClass = ref('');

const searchQuery = ref('');
const searchDob = ref('');

const currentPage = ref(1);
const currentLimit = ref(10);

const selectedUser = ref(null);
const isProcessing = ref(false);

const showDepositModal = ref(false);
const depositAmount = ref(0);

const showWithdrawModal = ref(false);
const withdrawAmount = ref(0);
const penaltyPercent = ref(0);

const showHistoryModal = ref(false);
const historyData = ref([]);
const historyLoading = ref(false);
const historyFilters = ref({
    type: '',
    start_date: ''
});
const historyMeta = ref({
    page: 1,
    limit: 5,
    total: 0,
    total_pages: 1,
    has_next: false,
    has_prev: false
});

const showReceiptModal = ref(false);
const printMode = ref('');
const receiptData = ref(null);

const formatRupiah = (angka) => {
    return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0
    }).format(angka);
};

const formatNumberInput = (value) => {
    if (!value && value !== 0) return '';
    const num = parseInt(String(value).replace(/\D/g, ''), 10);
    if (isNaN(num)) return '';
    return new Intl.NumberFormat('id-ID').format(num);
};

const depositAmountFormatted = computed(() => formatNumberInput(depositAmount.value));
const withdrawAmountFormatted = computed(() => formatNumberInput(withdrawAmount.value));

const onDepositInput = (e) => {
    const raw = e.target.value.replace(/\D/g, '');
    depositAmount.value = raw ? parseInt(raw, 10) : 0;
};

const onWithdrawInput = (e) => {
    const raw = e.target.value.replace(/\D/g, '');
    withdrawAmount.value = raw ? parseInt(raw, 10) : 0;
};

const onlyNumberKey = (e) => {
    if (!/[0-9]/.test(e.key)) {
        e.preventDefault();
    }
};

const formatTanggalIndo = (dateString) => {
    if (!dateString) return '-';
    const date = new Date(dateString);
    return date.toLocaleDateString('id-ID', {
        day: 'numeric',
        month: 'long',
        year: 'numeric'
    });
};

const formatDateTimeIndo = (dateString) => {
    if (!dateString) return '-';
    const date = new Date(dateString);
    return date.toLocaleDateString('id-ID', {
        day: 'numeric',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
    });
};

const showMessage = (msg, type = 'success') => {
    globalMessage.value = msg;
    messageClass.value = type === 'success'
        ? 'text-green-700 bg-green-100'
        : 'text-red-700 bg-red-100';
    setTimeout(() => {
        globalMessage.value = '';
    }, 5000);
};

const fetchBalances = async () => {
    isLoading.value = true;
    try {
        let url;

        if (searchQuery.value || searchDob.value) {
            const searchUrl = new URL(`${baseUrl}/api/users/search`);
            if (searchQuery.value) {
                searchUrl.searchParams.append('username', searchQuery.value);
            }
            if (searchDob.value) {
                const [year, month, day] = searchDob.value.split('-');
                searchUrl.searchParams.append('dob', `${day}-${month}-${year}`);
            }
            url = searchUrl.toString();
        } else {
            url = `${baseUrl}/api/admin/balances?page=${currentPage.value}&limit=${currentLimit.value}`;
        }

        const response = await fetch(url, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
            }
        });

        if (response.status === 404) {
            balances.value = [];
            isLoading.value = false;
            return;
        }

        const data = await response.json();
        if (!response.ok) throw new Error(data.message || 'Gagal mengambil data');

        balances.value = data.data || (Array.isArray(data) ? data : []);

        if (data.page) {
            currentPage.value = data.page;
        }

    } catch (error) {
        showMessage(error.message, 'error');
        router.push({
            name: "Login"
        })
    } finally {
        isLoading.value = false;
    }
};

let searchTimeout;
watch([searchQuery, searchDob], () => {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(() => {
        currentPage.value = 1;
        fetchBalances();
    }, 500);
});

const resetSearch = () => {
    searchQuery.value = '';
    searchDob.value = '';
};

const nextPage = () => {
    currentPage.value++;
    fetchBalances();
};

const prevPage = () => {
    if (currentPage.value > 1) {
        currentPage.value--;
        fetchBalances();
    }
};

const closeModal = () => {
    showDepositModal.value = false;
    showWithdrawModal.value = false;
    showHistoryModal.value = false;
    showReceiptModal.value = false;
    selectedUser.value = null;
    depositAmount.value = 0;
    withdrawAmount.value = 0;
    penaltyPercent.value = 0;
};

const openDepositModal = (user) => {
    selectedUser.value = user;
    depositAmount.value = 0;
    showDepositModal.value = true;
};

const openWithdrawModal = (user) => {
    selectedUser.value = user;
    withdrawAmount.value = 0;
    penaltyPercent.value = 0;
    showWithdrawModal.value = true;
};

const openHistoryModal = (user) => {
    selectedUser.value = user;
    historyFilters.value = { type: '', start_date: '' };
    historyMeta.value.page = 1;
    showHistoryModal.value = true;
    fetchHistory();
};

const fetchHistory = async () => {
    historyLoading.value = true;
    try {
        const params = new URLSearchParams({
            page: historyMeta.value.page,
            limit: historyMeta.value.limit,
            search: selectedUser.value.username
        });

        if (historyFilters.value.type) {
            params.append('type', historyFilters.value.type);
        }
        if (historyFilters.value.start_date) {
            params.append('start_date', historyFilters.value.start_date);
        }

        const response = await fetch(`${baseUrl}/api/history?${params.toString()}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
            }
        });

        if (!response.ok) throw new Error('Gagal mengambil riwayat transaksi');

        const result = await response.json();
        historyData.value = result.data || [];
        historyMeta.value = {
            page: result.page,
            limit: result.limit,
            total: result.total,
            total_pages: result.total_pages,
            has_next: result.has_next,
            has_prev: result.has_prev
        };
    } catch (error) {
        showMessage(error.message, 'error');
    } finally {
        historyLoading.value = false;
    }
};

const resetAndFetchHistory = () => {
    historyMeta.value.page = 1;
    fetchHistory();
};

const clearHistoryFilters = () => {
    historyFilters.value = { type: '', start_date: '' };
    resetAndFetchHistory();
};

const nextHistoryPage = () => {
    if (historyMeta.value.has_next) {
        historyMeta.value.page++;
        fetchHistory();
    }
};

const prevHistoryPage = () => {
    if (historyMeta.value.has_prev) {
        historyMeta.value.page--;
        fetchHistory();
    }
};

const submitDeposit = async () => {
    if (!depositAmount.value || depositAmount.value <= 0) {
        showMessage('Nominal setoran harus lebih dari 0', 'error');
        return;
    }

    isProcessing.value = true;
    try {
        const response = await fetch(`${baseUrl}/api/deposit`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
            },
            body: JSON.stringify({
                user_id: selectedUser.value.user_id,
                amount: depositAmount.value
            })
        });
        const result = await response.json();
        if (!response.ok) throw new Error(result.message || 'Gagal melakukan setoran');

        const newBalance = Number(selectedUser.value.balance) + Number(depositAmount.value);

        receiptData.value = {
            type: 'deposit',
            user: selectedUser.value.full_name || selectedUser.value.username,
            amount: depositAmount.value,
            date: new Date().toISOString(),
            finalBalance: newBalance
        };

        const userIndex = balances.value.findIndex(u => u.user_id === selectedUser.value.user_id);
        if (userIndex !== -1) {
            balances.value[userIndex].balance = newBalance;
        }

        showDepositModal.value = false;
        showReceiptModal.value = true;

        showMessage(`Setoran senilai ${formatRupiah(depositAmount.value)} berhasil ditambahkan!`, 'success');

        fetchBalances();
    } catch (error) {
        showMessage(error.message, 'error');
    } finally {
        isProcessing.value = false;
    }
};

const submitWithdraw = async () => {
    if (!withdrawAmount.value || withdrawAmount.value <= 0) {
        showMessage('Nominal penarikan harus lebih dari 0', 'error');
        return;
    }
    if (withdrawAmount.value > selectedUser.value.balance) {
        showMessage('Saldo tidak mencukupi untuk penarikan ini', 'error');
        return;
    }

    isProcessing.value = true;
    try {
        const response = await fetch(`${baseUrl}/api/withdraw`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
            },
            body: JSON.stringify({
                user_id: selectedUser.value.user_id,
                amount: withdrawAmount.value,
                penalty_percent: penaltyPercent.value
            })
        });
        const result = await response.json();
        if (!response.ok) throw new Error(result.message || 'Gagal melakukan penarikan');

        const penaltyAmount = Math.floor(withdrawAmount.value * (penaltyPercent.value / 100));
        const newBalance = Number(selectedUser.value.balance) - Number(withdrawAmount.value);

        receiptData.value = {
            type: 'withdraw',
            user: selectedUser.value.full_name || selectedUser.value.username,
            amount: withdrawAmount.value,
            penaltyPercent: penaltyPercent.value,
            penaltyAmount: penaltyAmount,
            receivedAmount: withdrawAmount.value - penaltyAmount,
            date: new Date().toISOString(),
            finalBalance: newBalance
        };

        const userIndex = balances.value.findIndex(u => u.user_id === selectedUser.value.user_id);
        if (userIndex !== -1) {
            balances.value[userIndex].balance = newBalance;
        }

        showWithdrawModal.value = false;
        showReceiptModal.value = true;

        showMessage(`Penarikan senilai ${formatRupiah(withdrawAmount.value)} berhasil!`, 'success');

        fetchBalances();
    } catch (error) {
        showMessage(error.message, 'error');
    } finally {
        isProcessing.value = false;
    }
};

const reprintTransaction = (history) => {
    let penaltyPct = 0;
    if (history.type === 'withdraw' && history.penalty_amount > 0) {
        penaltyPct = Math.round((history.penalty_amount / history.amount) * 100);
    }

    receiptData.value = {
        type: history.type,
        user: selectedUser.value.full_name || selectedUser.value.username,
        amount: history.amount,
        penaltyPercent: penaltyPct,
        penaltyAmount: history.penalty_amount || 0,
        receivedAmount: history.type === 'withdraw' ? (history.amount - (history.penalty_amount || 0)) : 0,
        date: history.created_at,
        finalBalance: history.balance || selectedUser.value.balance
    };

    printMode.value = 'receipt';
    nextTick(() => {
        window.print();
    });
};

const printReceiptAction = () => {
    printMode.value = 'receipt';
    nextTick(() => {
        window.print();
    });
};

const printHistoryAction = () => {
    printMode.value = 'history';
    nextTick(() => {
        window.print();
    });
};

onMounted(() => {
    fetchBalances();
});
</script>

<style>
@media print {
    body * {
        visibility: hidden;
    }

    #print-area,
    #print-area * {
        visibility: visible;
    }

    #print-area {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        margin: 0;
        padding: 0;
        background-color: white !important;
        color: black !important;
    }

    @page {
        size: A6 portrait;
        margin: 5mm;
    }

    * {
        -webkit-print-color-adjust: exact !important;
        print-color-adjust: exact !important;
    }
}
</style>