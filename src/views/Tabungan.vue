<template>
    <div>
        <div class="print:hidden">
            <div class="p-4 sm:p-6 lg:p-8 w-full max-w-screen-2xl mx-auto">

                <!-- Page header -->
                <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                    <div>
                        <h1 class="text-xl sm:text-2xl font-bold text-gray-800 dark:text-white">Manajemen Tabungan</h1>
                        <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">Kelola saldo, setoran, dan penarikan
                            nasabah</p>
                    </div>
                </div>

                <!-- Global message -->
                <transition name="fade">
                    <div v-if="globalMessage" :class="globalMessageIsError
                        ? 'text-red-800 dark:text-red-300 bg-red-50 dark:bg-red-900/30 border-red-200 dark:border-red-800'
                        : 'text-green-800 dark:text-green-300 bg-green-50 dark:bg-green-900/30 border-green-200 dark:border-green-800'"
                        class="mb-6 p-4 text-sm rounded-xl border flex items-center gap-3">
                        <Icon
                            :icon="globalMessageIsError ? 'heroicons:exclamation-triangle-solid' : 'heroicons:check-circle-solid'"
                            :class="globalMessageIsError ? 'text-red-500' : 'text-green-500'" class="w-5 h-5 shrink-0" />
                        <span class="flex-1">{{ globalMessage }}</span>
                        <button @click="globalMessage = ''"
                            class="shrink-0 p-1 rounded-lg hover:bg-black/5 dark:hover:bg-white/10 transition-colors">
                            <Icon icon="heroicons:x-mark" class="w-4 h-4" />
                        </button>
                    </div>
                </transition>

                <!-- Filter card -->
                <div
                    class="bg-white dark:bg-gray-800 p-4 sm:p-5 rounded-2xl shadow-sm mb-6 border border-gray-100 dark:border-gray-700">
                    <div class="grid grid-cols-1 sm:grid-cols-[1fr_auto] gap-3 sm:gap-4 items-end">
                        <div>
                            <label class="block text-xs font-semibold text-gray-500 dark:text-gray-400 mb-1.5">Filter
                                Tanggal Lahir</label>
                            <div class="relative">
                                <Icon icon="heroicons:calendar-days"
                                    class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400 pointer-events-none" />
                                <input v-model="searchDob" type="date"
                                    class="w-full pl-9 pr-3 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
                            </div>
                        </div>
                        <button @click="resetSearch" v-if="searchQuery || searchDob"
                            class="inline-flex items-center justify-center gap-1.5 bg-gray-100 hover:bg-gray-200 text-gray-700 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white px-4 py-2.5 rounded-xl text-sm font-medium transition-colors">
                            <Icon icon="heroicons:x-mark" class="w-4 h-4" />
                            Reset Pencarian
                        </button>
                    </div>
                </div>

                <!-- Loading skeleton -->
                <div v-if="isLoading" class="space-y-3">
                    <div v-for="n in 5" :key="n"
                        class="bg-white dark:bg-gray-800 rounded-2xl border border-gray-100 dark:border-gray-700 p-4 flex items-center gap-4 animate-pulse">
                        <div class="w-11 h-11 rounded-full bg-gray-200 dark:bg-gray-700 shrink-0"></div>
                        <div class="flex-1 space-y-2">
                            <div class="h-3.5 bg-gray-200 dark:bg-gray-700 rounded-full w-1/3"></div>
                            <div class="h-3 bg-gray-100 dark:bg-gray-700/60 rounded-full w-1/2"></div>
                        </div>
                        <div class="h-6 bg-gray-200 dark:bg-gray-700 rounded-full w-24 hidden sm:block"></div>
                    </div>
                </div>

                <template v-else>
                    <!-- Empty state -->
                    <div v-if="balances.length === 0"
                        class="text-center py-16 bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700">
                        <div
                            class="w-16 h-16 mx-auto mb-4 rounded-full bg-gray-100 dark:bg-gray-700 flex items-center justify-center">
                            <Icon icon="heroicons:banknotes" class="w-8 h-8 text-gray-400" />
                        </div>
                        <p class="text-gray-600 dark:text-gray-300 font-medium">Tidak ada data tabungan ditemukan</p>
                        <p class="text-sm text-gray-400 dark:text-gray-500 mt-1">Coba ubah filter pencarian Anda.</p>
                    </div>

                    <template v-else>
                        <!-- Desktop table (lg and up) -->
                        <div
                            class="hidden lg:block bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 overflow-hidden">
                            <div class="overflow-x-auto">
                                <table class="min-w-full divide-y divide-gray-100 dark:divide-gray-700">
                                    <thead class="bg-gray-50/80 dark:bg-gray-700/50">
                                        <tr>
                                            <th scope="col"
                                                class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Nasabah</th>
                                            <th scope="col"
                                                class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Tanggal Lahir</th>
                                            <th scope="col"
                                                class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Alamat</th>
                                            <th scope="col"
                                                class="px-6 py-3.5 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Total Saldo</th>
                                            <th scope="col"
                                                class="px-6 py-3.5 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Aksi Transaksi</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y divide-gray-100 dark:divide-gray-700">
                                        <tr v-for="item in balances" :key="item.user_id"
                                            class="hover:bg-indigo-50/40 dark:hover:bg-gray-700/50 transition-colors">
                                            <td class="px-6 py-4">
                                                <div class="flex items-center gap-3">
                                                    <div :class="avatarClass(item.username)"
                                                        class="w-10 h-10 rounded-full flex items-center justify-center text-white text-sm font-bold shrink-0">
                                                        {{ getInitials(item.full_name || item.username) }}
                                                    </div>
                                                    <div class="min-w-0">
                                                        <p
                                                            class="text-sm font-semibold text-gray-800 dark:text-white truncate">
                                                            {{ item.full_name || '-' }}</p>
                                                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">
                                                            @{{ item.username || '-' }}</p>
                                                    </div>
                                                </div>
                                            </td>
                                            <td
                                                class="px-6 py-4 whitespace-nowrap text-sm text-gray-600 dark:text-gray-300">
                                                {{ formatTanggalIndo(item.dob) }}</td>
                                            <td class="px-6 py-4 text-sm text-gray-600 dark:text-gray-300 max-w-[220px] truncate"
                                                :title="item.address">{{ item.address || '-' }}</td>
                                            <td class="px-6 py-4 whitespace-nowrap text-right">
                                                <span
                                                    class="inline-block px-3 py-1 rounded-full text-sm font-bold text-emerald-700 dark:text-emerald-300 bg-emerald-50 dark:bg-emerald-900/30 tabular-nums">
                                                    {{ formatRupiah(item.balance || 0) }}</span>
                                            </td>
                                            <td class="px-6 py-4 whitespace-nowrap text-right">
                                                <div class="inline-flex items-center gap-2">
                                                    <button @click="openDepositModal(item)"
                                                        class="inline-flex items-center gap-1.5 text-emerald-700 dark:text-emerald-400 bg-emerald-50 dark:bg-emerald-900/30 hover:bg-emerald-100 dark:hover:bg-emerald-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                                                        <Icon icon="heroicons:arrow-down-tray" class="w-4 h-4" />
                                                        Setor
                                                    </button>
                                                    <button @click="openWithdrawModal(item)"
                                                        class="inline-flex items-center gap-1.5 text-orange-700 dark:text-orange-400 bg-orange-50 dark:bg-orange-900/30 hover:bg-orange-100 dark:hover:bg-orange-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                                                        <Icon icon="heroicons:arrow-up-tray" class="w-4 h-4" />
                                                        Tarik
                                                    </button>
                                                    <button @click="openHistoryModal(item)"
                                                        class="inline-flex items-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                                                        <Icon icon="heroicons:clock" class="w-4 h-4" />
                                                        Riwayat
                                                    </button>
                                                </div>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <!-- Mobile / tablet cards (below lg) -->
                        <div class="lg:hidden space-y-3">
                            <div v-for="item in balances" :key="item.user_id"
                                class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 p-4">
                                <div class="flex items-center gap-3 mb-3">
                                    <div :class="avatarClass(item.username)"
                                        class="w-11 h-11 rounded-full flex items-center justify-center text-white text-sm font-bold shrink-0">
                                        {{ getInitials(item.full_name || item.username) }}
                                    </div>
                                    <div class="min-w-0 flex-1">
                                        <p class="text-sm font-semibold text-gray-800 dark:text-white truncate">{{
                                            item.full_name || '-' }}</p>
                                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">@{{ item.username
                                            || '-' }}</p>
                                    </div>
                                </div>
                                <div
                                    class="flex items-center justify-between bg-emerald-50 dark:bg-emerald-900/20 rounded-xl px-4 py-3 mb-3">
                                    <span class="text-xs font-medium text-emerald-700 dark:text-emerald-400">Total
                                        Saldo</span>
                                    <span
                                        class="text-base font-bold text-emerald-700 dark:text-emerald-300 tabular-nums">{{
                                            formatRupiah(item.balance || 0) }}</span>
                                </div>
                                <div class="grid grid-cols-2 gap-x-4 gap-y-2 text-sm mb-3">
                                    <div class="flex items-center gap-2 text-gray-600 dark:text-gray-300 min-w-0">
                                        <Icon icon="heroicons:cake" class="w-4 h-4 text-gray-400 shrink-0" />
                                        <span class="truncate">{{ formatTanggalIndo(item.dob) }}</span>
                                    </div>
                                    <div class="flex items-center gap-2 text-gray-600 dark:text-gray-300 min-w-0">
                                        <Icon icon="heroicons:map-pin" class="w-4 h-4 text-gray-400 shrink-0" />
                                        <span class="truncate">{{ item.address || '-' }}</span>
                                    </div>
                                </div>
                                <div class="grid grid-cols-3 gap-2 pt-3 border-t border-gray-100 dark:border-gray-700">
                                    <button @click="openDepositModal(item)"
                                        class="inline-flex items-center justify-center gap-1.5 text-emerald-700 dark:text-emerald-400 bg-emerald-50 dark:bg-emerald-900/30 hover:bg-emerald-100 dark:hover:bg-emerald-900/60 px-2 py-2 rounded-xl text-xs sm:text-sm font-semibold transition-colors">
                                        <Icon icon="heroicons:arrow-down-tray" class="w-4 h-4" />
                                        Setor
                                    </button>
                                    <button @click="openWithdrawModal(item)"
                                        class="inline-flex items-center justify-center gap-1.5 text-orange-700 dark:text-orange-400 bg-orange-50 dark:bg-orange-900/30 hover:bg-orange-100 dark:hover:bg-orange-900/60 px-2 py-2 rounded-xl text-xs sm:text-sm font-semibold transition-colors">
                                        <Icon icon="heroicons:arrow-up-tray" class="w-4 h-4" />
                                        Tarik
                                    </button>
                                    <button @click="openHistoryModal(item)"
                                        class="inline-flex items-center justify-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-2 py-2 rounded-xl text-xs sm:text-sm font-semibold transition-colors">
                                        <Icon icon="heroicons:clock" class="w-4 h-4" />
                                        Riwayat
                                    </button>
                                </div>
                            </div>
                        </div>
                    </template>

                    <!-- Pagination -->
                    <div v-if="balances.length > 0"
                        class="mt-4 px-4 sm:px-6 py-3 bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 flex items-center justify-between gap-3">
                        <button @click="prevPage" :disabled="currentPage === 1"
                            class="inline-flex items-center gap-1.5 px-3 sm:px-4 py-2 border border-gray-200 dark:border-gray-600 rounded-xl text-sm font-medium text-gray-700 dark:text-white hover:bg-gray-50 dark:hover:bg-gray-700 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
                            <Icon icon="heroicons:chevron-left" class="w-4 h-4" />
                            <span class="hidden sm:inline">Sebelumnya</span>
                        </button>
                        <span class="text-sm text-gray-600 dark:text-gray-300">Halaman <strong>{{ currentPage
                        }}</strong> dari <strong>{{ totalPages }}</strong></span>
                        <button @click="nextPage" :disabled="currentPage >= totalPages"
                            class="inline-flex items-center gap-1.5 px-3 sm:px-4 py-2 border border-gray-200 dark:border-gray-600 rounded-xl text-sm font-medium text-gray-700 dark:text-white hover:bg-gray-50 dark:hover:bg-gray-700 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
                            <span class="hidden sm:inline">Selanjutnya</span>
                            <Icon icon="heroicons:chevron-right" class="w-4 h-4" />
                        </button>
                    </div>
                </template>
            </div>

            <!-- Deposit modal -->
            <div v-if="showDepositModal"
                class="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
                @click.self="closeModal">
                <div
                    class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-md max-h-[92vh] flex flex-col overflow-hidden">
                    <div
                        class="px-5 sm:px-6 py-4 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center shrink-0">
                        <div class="flex items-center gap-3">
                            <div
                                class="w-9 h-9 rounded-xl bg-emerald-50 dark:bg-emerald-900/40 flex items-center justify-center text-emerald-600 dark:text-emerald-400">
                                <Icon icon="heroicons:arrow-down-tray" class="w-5 h-5" />
                            </div>
                            <h2 class="text-lg font-bold text-gray-800 dark:text-white">Setor Tabungan</h2>
                        </div>
                        <button @click="closeModal"
                            class="p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors">
                            <Icon icon="heroicons:x-mark" class="w-5 h-5" />
                        </button>
                    </div>
                    <div class="p-5 sm:p-6 overflow-y-auto">
                        <div
                            class="flex items-center gap-3 mb-4 p-3 bg-gray-50 dark:bg-gray-700/50 rounded-xl border border-gray-100 dark:border-gray-700">
                            <div :class="avatarClass(selectedUser?.username)"
                                class="w-9 h-9 rounded-full flex items-center justify-center text-white text-xs font-bold shrink-0">
                                {{ getInitials(selectedUser?.full_name || selectedUser?.username) }}
                            </div>
                            <p class="text-sm font-semibold text-gray-800 dark:text-white truncate">{{
                                selectedUser?.full_name || selectedUser?.username }}</p>
                        </div>
                        <form @submit.prevent="submitDeposit" class="space-y-4">
                            <div>
                                <label
                                    class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Nominal
                                    Setoran</label>
                                <div class="relative">
                                    <span
                                        class="absolute left-3.5 top-1/2 -translate-y-1/2 text-sm font-semibold text-gray-500 dark:text-gray-400">Rp</span>
                                    <input :value="depositAmountFormatted" @input="onDepositInput"
                                        @keypress="onlyNumberKey" type="text" inputmode="numeric" placeholder="0"
                                        required
                                        class="w-full pl-11 pr-3.5 py-3 text-lg font-semibold border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-emerald-500/40 focus:border-emerald-500 focus:bg-white dark:focus:bg-gray-900 text-right tabular-nums transition-all" />
                                </div>
                                <p v-if="depositAmount" class="mt-1.5 text-xs text-gray-500 dark:text-gray-400">
                                    {{ formatRupiah(depositAmount) }}
                                </p>
                            </div>
                            <div
                                class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3 pt-4 border-t border-gray-100 dark:border-gray-700 mt-6">
                                <button type="button" @click="closeModal"
                                    class="px-4 py-2.5 text-sm font-medium text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-xl transition-colors">Batal</button>
                                <button type="submit" :disabled="isProcessing || !depositAmount"
                                    class="inline-flex items-center justify-center gap-2 bg-emerald-600 hover:bg-emerald-700 text-white px-6 py-2.5 rounded-xl text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
                                    <Icon v-if="isProcessing" icon="heroicons:arrow-path" class="w-4 h-4 animate-spin" />
                                    {{ isProcessing ? 'Memproses...' : 'Simpan Setoran' }}
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>

            <!-- Withdraw modal -->
            <div v-if="showWithdrawModal"
                class="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
                @click.self="closeModal">
                <div
                    class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-md max-h-[92vh] flex flex-col overflow-hidden">
                    <div
                        class="px-5 sm:px-6 py-4 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center shrink-0">
                        <div class="flex items-center gap-3">
                            <div
                                class="w-9 h-9 rounded-xl bg-orange-50 dark:bg-orange-900/40 flex items-center justify-center text-orange-600 dark:text-orange-400">
                                <Icon icon="heroicons:arrow-up-tray" class="w-5 h-5" />
                            </div>
                            <h2 class="text-lg font-bold text-gray-800 dark:text-white">Tarik Tabungan</h2>
                        </div>
                        <button @click="closeModal"
                            class="p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors">
                            <Icon icon="heroicons:x-mark" class="w-5 h-5" />
                        </button>
                    </div>
                    <div class="p-5 sm:p-6 overflow-y-auto">
                        <div
                            class="flex items-center justify-between gap-3 mb-4 p-3 bg-gray-50 dark:bg-gray-700/50 rounded-xl border border-gray-100 dark:border-gray-700">
                            <div class="flex items-center gap-3 min-w-0">
                                <div :class="avatarClass(selectedUser?.username)"
                                    class="w-9 h-9 rounded-full flex items-center justify-center text-white text-xs font-bold shrink-0">
                                    {{ getInitials(selectedUser?.full_name || selectedUser?.username) }}
                                </div>
                                <p class="text-sm font-semibold text-gray-800 dark:text-white truncate">{{
                                    selectedUser?.full_name || selectedUser?.username }}</p>
                            </div>
                            <div class="text-right shrink-0">
                                <p class="text-[10px] uppercase tracking-wide text-gray-400">Saldo</p>
                                <p class="text-sm font-bold text-emerald-600 dark:text-emerald-400 tabular-nums">{{
                                    formatRupiah(selectedUser?.balance || 0) }}</p>
                            </div>
                        </div>
                        <form @submit.prevent="submitWithdraw" class="space-y-4">
                            <div>
                                <label
                                    class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Nominal
                                    Penarikan</label>
                                <div class="relative">
                                    <span
                                        class="absolute left-3.5 top-1/2 -translate-y-1/2 text-sm font-semibold text-gray-500 dark:text-gray-400">Rp</span>
                                    <input :value="withdrawAmountFormatted" @input="onWithdrawInput"
                                        @keypress="onlyNumberKey" type="text" inputmode="numeric" placeholder="0"
                                        required
                                        class="w-full pl-11 pr-3.5 py-3 text-lg font-semibold border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-orange-500/40 focus:border-orange-500 focus:bg-white dark:focus:bg-gray-900 text-right tabular-nums transition-all" />
                                </div>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Skema
                                    Potongan Otomatis</label>
                                <div
                                    class="w-full px-4 py-3 border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white">
                                    <p class="text-sm font-medium text-gray-800 dark:text-gray-100">
                                        {{ withdrawAmount < 1000000 ? 'Di bawah Rp1.000.000: potongan Rp5.000' : 'Rp1.000.000 ke atas: potongan 1%' }}
                                    </p>
                                    <p v-if="withdrawAmount" class="mt-1 text-xs text-gray-500 dark:text-gray-400">
                                        Persentase efektif: {{ formatPercent(effectivePenaltyPercent) }}%
                                    </p>
                                </div>

                                <div v-if="withdrawAmount"
                                    class="mt-3 p-4 bg-orange-50/60 dark:bg-gray-700 rounded-xl border border-orange-100 dark:border-gray-600">
                                    <div class="flex justify-between text-sm mb-1">
                                        <span class="text-gray-600 dark:text-gray-400">Potongan Penalti ({{
                                            formatPercent(effectivePenaltyPercent) }}%):</span>
                                        <span class="text-red-600 dark:text-red-400 font-medium tabular-nums">-{{
                                            formatRupiah(withdrawPenaltyAmount) }}</span>
                                    </div>
                                    <div
                                        class="flex justify-between text-sm font-bold border-t border-orange-200 dark:border-gray-600 pt-2 mt-2">
                                        <span class="text-gray-800 dark:text-gray-200">Uang Diterima:</span>
                                        <span class="text-emerald-600 dark:text-emerald-400 tabular-nums">{{
                                            formatRupiah(withdrawReceivedAmount) }}</span>
                                    </div>
                                </div>
                            </div>
                            <div
                                class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3 pt-4 border-t border-gray-100 dark:border-gray-700 mt-6">
                                <button type="button" @click="closeModal"
                                    class="px-4 py-2.5 text-sm font-medium text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-xl transition-colors">Batal</button>
                                <button type="submit" :disabled="isProcessing || !withdrawAmount"
                                    class="inline-flex items-center justify-center gap-2 bg-orange-600 hover:bg-orange-700 text-white px-6 py-2.5 rounded-xl text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
                                    <Icon v-if="isProcessing" icon="heroicons:arrow-path" class="w-4 h-4 animate-spin" />
                                    {{ isProcessing ? 'Memproses...' : 'Tarik Dana' }}
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>

            <!-- History modal -->
            <div v-if="showHistoryModal"
                class="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
                @click.self="closeModal">
                <div
                    class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-5xl overflow-hidden relative flex flex-col max-h-[92vh]">
                    <div
                        class="px-5 sm:px-6 py-4 border-b border-gray-100 dark:border-gray-700 flex flex-col sm:flex-row sm:justify-between sm:items-center gap-3 shrink-0">
                        <div class="flex items-center justify-between sm:justify-start gap-3 min-w-0">
                            <div class="flex items-center gap-3 min-w-0">
                                <div
                                    class="w-9 h-9 rounded-xl bg-indigo-50 dark:bg-indigo-900/40 flex items-center justify-center text-indigo-600 dark:text-indigo-400 shrink-0">
                                    <Icon icon="heroicons:clock" class="w-5 h-5" />
                                </div>
                                <h2 class="text-base sm:text-lg font-bold text-gray-800 dark:text-white truncate">
                                    Riwayat: {{ selectedUser?.full_name || selectedUser?.username }}</h2>
                            </div>
                            <button @click="closeModal"
                                class="sm:hidden p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors shrink-0">
                                <Icon icon="heroicons:x-mark" class="w-5 h-5" />
                            </button>
                        </div>
                        <div class="flex items-center gap-2 shrink-0">
                            <button @click="printHistoryAction"
                                class="flex-1 sm:flex-none inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-xl text-sm font-semibold transition-colors">
                                <Icon icon="heroicons:printer" class="w-4 h-4" />
                                Cetak Riwayat
                            </button>
                            <button @click="closeModal"
                                class="hidden sm:block p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors">
                                <Icon icon="heroicons:x-mark" class="w-5 h-5" />
                            </button>
                        </div>
                    </div>

                    <div
                        class="p-4 sm:p-5 shrink-0 bg-gray-50 dark:bg-gray-800 border-b border-gray-100 dark:border-gray-700">
                        <div class="grid grid-cols-1 sm:grid-cols-[1fr_1fr_auto] gap-3 items-end">
                            <div>
                                <label class="block text-xs font-semibold text-gray-500 dark:text-gray-400 mb-1.5">Tipe
                                    Transaksi</label>
                                <select v-model="historyFilters.type" @change="resetAndFetchHistory"
                                    class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-white dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 transition-all">
                                    <option value="">Semua</option>
                                    <option value="deposit">Setor (Deposit)</option>
                                    <option value="withdraw">Tarik (Withdraw)</option>
                                </select>
                            </div>
                            <div>
                                <label
                                    class="block text-xs font-semibold text-gray-500 dark:text-gray-400 mb-1.5">Mulai
                                    Tanggal</label>
                                <input v-model="historyFilters.start_date" type="date" @change="resetAndFetchHistory"
                                    class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-white dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 transition-all" />
                            </div>
                            <button @click="clearHistoryFilters" v-if="historyFilters.type || historyFilters.start_date"
                                class="inline-flex items-center justify-center gap-1.5 bg-gray-100 hover:bg-gray-200 text-gray-700 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white px-4 py-2.5 rounded-xl text-sm font-medium transition-colors">
                                <Icon icon="heroicons:x-mark" class="w-4 h-4" />
                                Reset
                            </button>
                        </div>
                    </div>

                    <div class="p-4 sm:p-6 overflow-y-auto flex-1">
                        <div v-if="historyLoading" class="space-y-3">
                            <div v-for="n in 4" :key="n"
                                class="h-12 bg-gray-100 dark:bg-gray-700/60 rounded-xl animate-pulse"></div>
                        </div>
                        <div v-else>
                            <!-- Desktop history table (sm and up) -->
                            <div class="hidden sm:block overflow-x-auto">
                                <table class="min-w-full divide-y divide-gray-100 dark:divide-gray-700">
                                    <thead class="bg-gray-50/80 dark:bg-gray-700/50">
                                        <tr>
                                            <th scope="col"
                                                class="px-4 py-3 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider rounded-l-lg">
                                                Tanggal</th>
                                            <th scope="col"
                                                class="px-4 py-3 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Tipe</th>
                                            <th scope="col"
                                                class="px-4 py-3 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Nominal</th>
                                            <th scope="col"
                                                class="px-4 py-3 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Penalti</th>
                                            <th scope="col"
                                                class="px-4 py-3 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                                                Total Akhir</th>
                                            <th scope="col"
                                                class="px-4 py-3 text-center text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider rounded-r-lg">
                                                Aksi</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y divide-gray-100 dark:divide-gray-700">
                                        <tr v-for="history in historyData" :key="history.id"
                                            class="hover:bg-indigo-50/40 dark:hover:bg-gray-700/50 transition-colors">
                                            <td
                                                class="px-4 py-3 whitespace-nowrap text-sm text-gray-600 dark:text-gray-300">
                                                {{ formatDateTimeIndo(history.created_at) }}</td>
                                            <td class="px-4 py-3 whitespace-nowrap text-sm">
                                                <span :class="history.type === 'deposit'
                                                    ? 'text-emerald-700 dark:text-emerald-300 bg-emerald-50 dark:bg-emerald-900/30'
                                                    : 'text-orange-700 dark:text-orange-300 bg-orange-50 dark:bg-orange-900/30'"
                                                    class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-semibold">
                                                    <Icon
                                                        :icon="history.type === 'deposit' ? 'heroicons:arrow-down-tray' : 'heroicons:arrow-up-tray'"
                                                        class="w-3.5 h-3.5" />
                                                    {{ history.type === 'deposit' ? 'Setor' : 'Tarik' }}
                                                </span>
                                            </td>
                                            <td
                                                class="px-4 py-3 whitespace-nowrap text-sm text-right font-medium text-gray-900 dark:text-white tabular-nums">
                                                {{ formatRupiah(history.amount) }}</td>
                                            <td
                                                class="px-4 py-3 whitespace-nowrap text-sm text-right text-red-500 tabular-nums">
                                                {{ history.penalty_amount > 0 ? '-' +
                                                    formatRupiah(history.penalty_amount) : '-' }}</td>
                                            <td
                                                class="px-4 py-3 whitespace-nowrap text-sm text-right font-bold text-gray-900 dark:text-white tabular-nums">
                                                {{ formatRupiah(history.final_amount) }}</td>
                                            <td class="px-4 py-3 whitespace-nowrap text-sm text-center">
                                                <button @click="reprintTransaction(history)"
                                                    class="inline-flex items-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                                                    <Icon icon="heroicons:printer" class="w-3.5 h-3.5" />
                                                    Cetak
                                                </button>
                                            </td>
                                        </tr>
                                        <tr v-if="historyData.length === 0">
                                            <td colspan="6"
                                                class="px-4 py-10 text-center text-gray-500 dark:text-gray-400">
                                                Tidak ada riwayat transaksi ditemukan.</td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>

                            <!-- Mobile history cards -->
                            <div class="sm:hidden space-y-3">
                                <div v-for="history in historyData" :key="history.id"
                                    class="border border-gray-100 dark:border-gray-700 rounded-xl p-3.5">
                                    <div class="flex items-center justify-between mb-2">
                                        <span :class="history.type === 'deposit'
                                            ? 'text-emerald-700 dark:text-emerald-300 bg-emerald-50 dark:bg-emerald-900/30'
                                            : 'text-orange-700 dark:text-orange-300 bg-orange-50 dark:bg-orange-900/30'"
                                            class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-semibold">
                                            <Icon
                                                :icon="history.type === 'deposit' ? 'heroicons:arrow-down-tray' : 'heroicons:arrow-up-tray'"
                                                class="w-3.5 h-3.5" />
                                            {{ history.type === 'deposit' ? 'Setor' : 'Tarik' }}
                                        </span>
                                        <span class="text-xs text-gray-400">{{ formatDateTimeIndo(history.created_at)
                                            }}</span>
                                    </div>
                                    <div class="space-y-1 text-sm">
                                        <div class="flex justify-between">
                                            <span class="text-gray-500 dark:text-gray-400">Nominal</span>
                                            <span class="font-medium text-gray-900 dark:text-white tabular-nums">{{
                                                formatRupiah(history.amount) }}</span>
                                        </div>
                                        <div v-if="history.penalty_amount > 0" class="flex justify-between">
                                            <span class="text-gray-500 dark:text-gray-400">Penalti</span>
                                            <span class="text-red-500 tabular-nums">-{{
                                                formatRupiah(history.penalty_amount) }}</span>
                                        </div>
                                        <div
                                            class="flex justify-between pt-1.5 mt-1.5 border-t border-gray-100 dark:border-gray-700">
                                            <span class="text-gray-500 dark:text-gray-400">Total Akhir</span>
                                            <span class="font-bold text-gray-900 dark:text-white tabular-nums">{{
                                                formatRupiah(history.final_amount) }}</span>
                                        </div>
                                    </div>
                                    <button @click="reprintTransaction(history)"
                                        class="mt-3 w-full inline-flex items-center justify-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-3 py-2 rounded-xl text-xs font-semibold transition-colors">
                                        <Icon icon="heroicons:printer" class="w-4 h-4" />
                                        Cetak Ulang
                                    </button>
                                </div>
                                <div v-if="historyData.length === 0"
                                    class="py-10 text-center text-gray-500 dark:text-gray-400 text-sm">
                                    Tidak ada riwayat transaksi ditemukan.</div>
                            </div>
                        </div>
                    </div>

                    <div
                        class="px-4 sm:px-6 py-3.5 border-t border-gray-100 dark:border-gray-700 flex justify-between items-center gap-3 bg-gray-50 dark:bg-gray-800 shrink-0">
                        <button @click="prevHistoryPage" :disabled="!historyMeta.has_prev"
                            class="inline-flex items-center gap-1.5 px-3 sm:px-4 py-2 border border-gray-200 dark:border-gray-600 rounded-xl text-sm font-medium text-gray-700 dark:text-white hover:bg-white dark:hover:bg-gray-700 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
                            <Icon icon="heroicons:chevron-left" class="w-4 h-4" />
                            <span class="hidden sm:inline">Sebelumnya</span>
                        </button>
                        <span class="text-xs sm:text-sm text-gray-600 dark:text-gray-300 text-center">Hal. {{
                            historyMeta.page }} / {{ historyMeta.total_pages }} <span
                                class="hidden sm:inline">(Total: {{ historyMeta.total }})</span></span>
                        <button @click="nextHistoryPage" :disabled="!historyMeta.has_next"
                            class="inline-flex items-center gap-1.5 px-3 sm:px-4 py-2 border border-gray-200 dark:border-gray-600 rounded-xl text-sm font-medium text-gray-700 dark:text-white hover:bg-white dark:hover:bg-gray-700 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
                            <span class="hidden sm:inline">Selanjutnya</span>
                            <Icon icon="heroicons:chevron-right" class="w-4 h-4" />
                        </button>
                    </div>
                </div>
            </div>

            <!-- Receipt modal -->
            <div v-if="showReceiptModal"
                class="fixed inset-0 z-[60] flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
                @click.self="closeModal">
                <div
                    class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-sm overflow-hidden text-center p-6">
                    <div
                        class="w-16 h-16 bg-emerald-100 dark:bg-emerald-900/50 text-emerald-600 dark:text-emerald-400 rounded-full flex items-center justify-center mx-auto mb-4">
                        <Icon icon="heroicons:check" class="w-8 h-8" />
                    </div>
                    <h2 class="text-xl font-bold text-gray-800 dark:text-white mb-2">Transaksi Berhasil!</h2>

                    <div v-if="receiptData"
                        class="bg-gray-50 dark:bg-gray-700 p-4 rounded-xl mb-6 text-left w-full text-sm border border-gray-100 dark:border-gray-600">
                        <div class="flex justify-between mb-2">
                            <span class="text-gray-600 dark:text-gray-400">Jenis Transaksi:</span>
                            <span class="font-semibold text-gray-800 dark:text-white">
                                {{ receiptData.type === 'deposit' ? 'Setor Tunai'
                                    : 'Tarik Tunai' }}</span>
                        </div>
                        <div class="flex justify-between mb-2">
                            <span class="text-gray-600 dark:text-gray-400">Nominal:</span>
                            <span
                                :class="receiptData.type === 'deposit' ? 'text-emerald-600 dark:text-emerald-400' : 'text-orange-600 dark:text-orange-400'"
                                class="font-semibold tabular-nums">
                                {{ receiptData.type === 'deposit' ? '+' : '-' }} {{ formatRupiah(receiptData.amount ||
                                    0) }}
                            </span>
                        </div>
                        <div class="flex justify-between pt-2 border-t border-gray-200 dark:border-gray-600">
                            <span class="text-gray-600 dark:text-gray-400">Total Saldo:</span>
                            <span class="font-bold text-lg text-gray-800 dark:text-white tabular-nums">{{
                                formatRupiah(receiptData.finalBalance || 0) }}</span>
                        </div>
                    </div>

                    <div class="flex flex-col-reverse sm:flex-row gap-3 justify-center">
                        <button @click="closeModal"
                            class="px-4 py-2.5 text-sm text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-xl transition-colors font-medium">Tutup</button>
                        <button @click="printReceiptAction"
                            class="inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2.5 rounded-xl text-sm font-semibold transition-colors">
                            <Icon icon="heroicons:printer" class="w-4 h-4" />
                            Cetak Struk
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div id="print-area" class="bg-white text-black font-sans w-full mx-auto">
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
                        <span>Potongan Penalti ({{ formatPercent(receiptData.penaltyPercent) }}%):</span>
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
import { ref, computed, watch, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { Icon } from '@iconify/vue';

const token = localStorage.getItem('token');
const baseUrl = 'https://alentest.my.id/tabungan';
const router = useRouter()

const balances = ref([]);
const isLoading = ref(true);
const globalMessage = ref('');
const globalMessageIsError = ref(false);

const searchQuery = ref('');
const searchDob = ref('');

const currentPage = ref(1);
const currentLimit = ref(10);
const totalPages = ref(1);
const totalItems = ref(0);

const selectedUser = ref(null);
const isProcessing = ref(false);

const showDepositModal = ref(false);
const depositAmount = ref(0);

const showWithdrawModal = ref(false);
const withdrawAmount = ref(0);

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

const avatarPalette = [
    'bg-gradient-to-br from-indigo-500 to-purple-500',
    'bg-gradient-to-br from-sky-500 to-blue-600',
    'bg-gradient-to-br from-emerald-500 to-teal-600',
    'bg-gradient-to-br from-amber-500 to-orange-600',
    'bg-gradient-to-br from-rose-500 to-pink-600',
    'bg-gradient-to-br from-violet-500 to-fuchsia-600'
];

const avatarClass = (seed) => {
    const str = String(seed || '');
    let hash = 0;
    for (let i = 0; i < str.length; i++) hash = (hash * 31 + str.charCodeAt(i)) >>> 0;
    return avatarPalette[hash % avatarPalette.length];
};

const getInitials = (name) => {
    if (!name) return '?';
    const parts = String(name).trim().split(/\s+/);
    if (parts.length === 1) return parts[0].substring(0, 2).toUpperCase();
    return (parts[0][0] + parts[1][0]).toUpperCase();
};

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

const formatPercent = (value) => {
    if (!Number.isFinite(value)) return '0';
    return new Intl.NumberFormat('id-ID', {
        minimumFractionDigits: 0,
        maximumFractionDigits: 2
    }).format(value);
};

const depositAmountFormatted = computed(() => formatNumberInput(depositAmount.value));
const withdrawAmountFormatted = computed(() => formatNumberInput(withdrawAmount.value));
const effectivePenaltyPercent = computed(() => {
    if (!withdrawAmount.value || withdrawAmount.value <= 0) return 0;
    if (withdrawAmount.value < 1000000) return (5000 / withdrawAmount.value) * 100;
    return 1;
});
const withdrawPenaltyAmount = computed(() => {
    if (!withdrawAmount.value || withdrawAmount.value <= 0) return 0;
    if (withdrawAmount.value < 1000000) return 5000;
    return Math.floor(withdrawAmount.value * 0.01);
});
const withdrawReceivedAmount = computed(() => {
    if (!withdrawAmount.value || withdrawAmount.value <= 0) return 0;
    return withdrawAmount.value - withdrawPenaltyAmount.value;
});

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
    globalMessageIsError.value = type !== 'success';
    setTimeout(() => {
        globalMessage.value = '';
    }, 5000);
};

const fetchBalances = async () => {
    isLoading.value = true;
    try {
        const url = new URL(`${baseUrl}/api/admin/balances`);

        url.searchParams.append('page', currentPage.value);
        url.searchParams.append('limit', currentLimit.value);

        if (searchQuery.value) {
            url.searchParams.append('search', searchQuery.value);
        }

        if (searchDob.value) {
            const [year, month, day] = searchDob.value.split('-');
            url.searchParams.append('dob', `${day}-${month}-${year}`);
        }

        const response = await fetch(url.toString(), {
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

        if (data.page) currentPage.value = data.page;
        if (data.total_pages) totalPages.value = data.total_pages;
        if (data.total) totalItems.value = data.total;

    } catch (error) {
        showMessage(error.message, 'error');
        router.push({
            name: "Login"
        });
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
    if (currentPage.value < totalPages.value) {
        currentPage.value++;
        fetchBalances();
    }
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
};

const openDepositModal = (user) => {
    selectedUser.value = user;
    depositAmount.value = 0;
    showDepositModal.value = true;
};

const openWithdrawModal = (user) => {
    selectedUser.value = user;
    withdrawAmount.value = 0;
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
    if (withdrawAmount.value <= withdrawPenaltyAmount.value) {
        showMessage('Nominal penarikan harus lebih besar dari potongan penalti', 'error');
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
                penalty_percent: effectivePenaltyPercent.value
            })
        });
        const result = await response.json();
        if (!response.ok) throw new Error(result.message || 'Gagal melakukan penarikan');

        const penaltyAmount = withdrawPenaltyAmount.value;
        const newBalance = Number(selectedUser.value.balance) - Number(withdrawAmount.value);

        receiptData.value = {
            type: 'withdraw',
            user: selectedUser.value.full_name || selectedUser.value.username,
            amount: withdrawAmount.value,
            penaltyPercent: effectivePenaltyPercent.value,
            penaltyAmount: penaltyAmount,
            receivedAmount: withdrawReceivedAmount.value,
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

const escapeHtml = (value) => String(value ?? '')
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');

const printHtml = (html) => {
    const iframe = document.createElement('iframe');
    iframe.style.position = 'fixed';
    iframe.style.right = '0';
    iframe.style.bottom = '0';
    iframe.style.width = '0';
    iframe.style.height = '0';
    iframe.style.border = '0';
    iframe.style.opacity = '0';
    document.body.appendChild(iframe);

    const printDocument = iframe.contentWindow.document;
    printDocument.open();
    printDocument.write(html);
    printDocument.close();

    const cleanup = () => {
        if (iframe.parentNode) iframe.remove();
    };

    setTimeout(() => {
        iframe.contentWindow.onafterprint = cleanup;
        iframe.contentWindow.focus();
        iframe.contentWindow.print();
        setTimeout(cleanup, 30000);
    }, 100);
};

const buildPrintPage = (content, pageSize = 'A6 portrait') => `
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Cetak</title>
    <style>
        @page { size: ${pageSize}; margin: 5mm; }
        * { box-sizing: border-box; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        body { margin: 0; font-family: Arial, sans-serif; color: #000; background: #fff; }
        .receipt { width: 100%; border: 1px solid #d1d5db; padding: 16px; border-radius: 4px; }
        .center { text-align: center; }
        .title { font-size: 18px; font-weight: 700; text-transform: uppercase; margin: 0; }
        .subtitle { font-size: 11px; color: #6b7280; margin: 4px 0 0; }
        .section { border-top: 1px solid #000; border-bottom: 1px solid #000; padding: 8px 0; margin: 14px 0; }
        .line { display: flex; justify-content: space-between; gap: 12px; margin: 5px 0; font-size: 12px; }
        .line strong { font-size: 14px; }
        .muted { color: #4b5563; }
        .divider { border-top: 1px dashed #9ca3af; padding-top: 6px; margin-top: 6px; }
        .solid-divider { border-top: 1px solid #000; padding-top: 8px; margin-top: 8px; }
        table { width: 100%; border-collapse: collapse; font-size: 10px; }
        th, td { border: 1px solid #9ca3af; padding: 5px; }
        th { background: #f3f4f6; text-align: left; }
        .right { text-align: right; }
        .footer { text-align: center; color: #6b7280; font-size: 10px; margin-top: 14px; }
    </style>
</head>
<body>${content}</body>
</html>`;

const buildReceiptPrintHtml = (data) => {
    const isWithdraw = data.type === 'withdraw';
    const penaltyRow = isWithdraw && Number(data.penaltyAmount) > 0
        ? `<div class="line muted"><span>Potongan Penalti (${escapeHtml(formatPercent(Number(data.penaltyPercent || 0)))}%):</span><span>-${escapeHtml(formatRupiah(Number(data.penaltyAmount || 0)))}</span></div>`
        : '';
    const receivedRow = isWithdraw
        ? `<div class="line divider"><strong>Uang Diterima:</strong><strong>${escapeHtml(formatRupiah(Number(data.receivedAmount || 0)))}</strong></div>`
        : '';

    return buildPrintPage(`
        <div class="receipt">
            <div class="center" style="border-bottom: 1px solid #d1d5db; padding-bottom: 12px;">
                <h2 class="title">${isWithdraw ? 'Bukti Penarikan' : 'Bukti Setoran'}</h2>
                <p class="subtitle">Manajemen Tabungan App</p>
            </div>
            <div style="margin-top: 12px;">
                <div class="line"><span class="muted">Tanggal:</span><span>${escapeHtml(formatDateTimeIndo(data.date))}</span></div>
                <div class="line"><span class="muted">Nama:</span><span>${escapeHtml(data.user || '-')}</span></div>
                <div class="line"><span class="muted">Tipe:</span><span>${isWithdraw ? 'Tarik Tunai' : 'Setor Tunai'}</span></div>
            </div>
            <div class="section">
                <div class="line"><strong>${isWithdraw ? 'Nominal Tarik:' : 'Nominal Setor:'}</strong><strong>${escapeHtml(formatRupiah(Number(data.amount || 0)))}</strong></div>
                ${penaltyRow}
                ${receivedRow}
                <div class="line solid-divider"><strong>Sisa Saldo:</strong><strong>${escapeHtml(formatRupiah(Number(data.finalBalance || 0)))}</strong></div>
            </div>
            <div class="footer">
                <p>Terima kasih telah menggunakan layanan kami.</p>
                <p>Struk ini adalah bukti transaksi yang sah.</p>
            </div>
        </div>
    `);
};

const buildHistoryPrintHtml = () => {
    const userName = selectedUser.value?.full_name || selectedUser.value?.username || '-';
    const rows = historyData.value.length
        ? historyData.value.map((history) => `
            <tr>
                <td>${escapeHtml(formatDateTimeIndo(history.created_at))}</td>
                <td>${history.type === 'deposit' ? 'Setor' : 'Tarik'}</td>
                <td class="right">${escapeHtml(formatRupiah(Number(history.amount || 0)))}</td>
                <td class="right"><strong>${escapeHtml(formatRupiah(Number(history.final_amount || 0)))}</strong></td>
            </tr>
        `).join('')
        : '<tr><td colspan="4" class="center">Tidak ada data transaksi.</td></tr>';

    return buildPrintPage(`
        <div class="center" style="border-bottom: 1px solid #000; padding-bottom: 12px; margin-bottom: 16px;">
            <h2 class="title">Riwayat Transaksi</h2>
            <p>Nama Nasabah: <strong>${escapeHtml(userName)}</strong></p>
            <p>Total Saldo Terakhir: <strong>${escapeHtml(formatRupiah(Number(selectedUser.value?.balance || 0)))}</strong></p>
            <p class="subtitle">Dicetak pada: ${escapeHtml(formatDateTimeIndo(new Date().toISOString()))}</p>
        </div>
        <table>
            <thead>
                <tr>
                    <th>Tgl & Waktu</th>
                    <th>Tipe</th>
                    <th class="right">Nominal</th>
                    <th class="right">Total Akhir</th>
                </tr>
            </thead>
            <tbody>${rows}</tbody>
        </table>
    `, 'A4 portrait');
};

const reprintTransaction = async (history) => {
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
    printHtml(buildReceiptPrintHtml(receiptData.value));
};

const printReceiptAction = async () => {
    if (!receiptData.value) {
        showMessage('Data struk belum tersedia untuk dicetak', 'error');
        return;
    }

    printMode.value = 'receipt';
    printHtml(buildReceiptPrintHtml(receiptData.value));
};

const printHistoryAction = async () => {
    if (!selectedUser.value) {
        showMessage('Data nasabah belum tersedia untuk dicetak', 'error');
        return;
    }

    printMode.value = 'history';
    printHtml(buildHistoryPrintHtml());
};

onMounted(() => {
    fetchBalances();
});
</script>

<style>
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

#print-area {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    transform: translateX(-200vw);
    visibility: hidden;
    pointer-events: none;
}

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
        display: block;
        width: 100%;
        margin: 0;
        padding: 0;
        transform: none;
        pointer-events: auto;
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
