<template>
  <div class="p-4 sm:p-6 lg:p-8 w-full max-w-screen-2xl mx-auto">

    <!-- Page header -->
    <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
      <div>
        <h1 class="text-xl sm:text-2xl font-bold text-gray-800 dark:text-white">Manajemen Pengguna</h1>
        <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">Kelola data nasabah dan akun pengguna</p>
      </div>
      <button @click="showAddModal = true"
        class="inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 active:scale-[0.98] text-white px-4 py-2.5 rounded-xl text-sm font-semibold shadow-sm shadow-indigo-200 dark:shadow-none transition-all w-full sm:w-auto">
        <Icon icon="heroicons:user-plus-solid" class="w-5 h-5" />
        Tambah Pengguna
      </button>
    </div>

    <!-- Success message -->
    <transition name="fade">
      <div v-if="globalSuccessMessage"
        class="mb-6 p-4 text-sm text-green-800 dark:text-green-300 bg-green-50 dark:bg-green-900/30 border border-green-200 dark:border-green-800 rounded-xl flex items-center gap-3">
        <Icon icon="heroicons:check-circle-solid" class="w-5 h-5 shrink-0 text-green-500" />
        <span class="flex-1">{{ globalSuccessMessage }}</span>
        <button @click="globalSuccessMessage = ''"
          class="shrink-0 p-1 rounded-lg hover:bg-green-100 dark:hover:bg-green-800/50 transition-colors">
          <Icon icon="heroicons:x-mark" class="w-4 h-4" />
        </button>
      </div>
    </transition>

    <!-- Filter card -->
    <div
      class="bg-white dark:bg-gray-800 p-4 sm:p-5 rounded-2xl shadow-sm mb-6 border border-gray-100 dark:border-gray-700">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-[1fr_1fr_auto] gap-3 sm:gap-4 items-end">
        <div>
          <label class="block text-xs font-semibold text-gray-500 dark:text-gray-400 mb-1.5">Cari Username</label>
          <div class="relative">
            <Icon icon="heroicons:magnifying-glass"
              class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400 pointer-events-none" />
            <input v-model="searchQuery.username" type="text" placeholder="Mulai ketik nama..."
              class="w-full pl-9 pr-3 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
          </div>
        </div>
        <div>
          <label class="block text-xs font-semibold text-gray-500 dark:text-gray-400 mb-1.5">Filter Tanggal
            Lahir</label>
          <div class="relative">
            <Icon icon="heroicons:calendar-days"
              class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400 pointer-events-none" />
            <input v-model="searchQuery.dob" type="date"
              class="w-full pl-9 pr-3 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
          </div>
        </div>
        <button @click="clearSearch" v-if="searchQuery.username || searchQuery.dob"
          class="inline-flex items-center justify-center gap-1.5 bg-gray-100 hover:bg-gray-200 text-gray-700 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white px-4 py-2.5 rounded-xl text-sm font-medium transition-colors">
          <Icon icon="heroicons:x-mark" class="w-4 h-4" />
          Hapus Filter
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
      </div>
    </div>

    <!-- Error -->
    <div v-else-if="errorMessage"
      class="p-4 text-sm text-red-700 dark:text-red-300 bg-red-50 dark:bg-red-900/30 border border-red-200 dark:border-red-800 rounded-xl mb-4 flex items-center gap-3">
      <Icon icon="heroicons:exclamation-triangle-solid" class="w-5 h-5 shrink-0 text-red-500" />
      {{ errorMessage }}
    </div>

    <!-- Empty state -->
    <div v-else-if="users.length === 0"
      class="text-center py-16 bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700">
      <div
        class="w-16 h-16 mx-auto mb-4 rounded-full bg-gray-100 dark:bg-gray-700 flex items-center justify-center">
        <Icon icon="heroicons:user-group" class="w-8 h-8 text-gray-400" />
      </div>
      <p class="text-gray-600 dark:text-gray-300 font-medium">Tidak ada pengguna ditemukan</p>
      <p class="text-sm text-gray-400 dark:text-gray-500 mt-1">Coba ubah kata kunci atau filter pencarian Anda.</p>
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
                  Pengguna</th>
                <th scope="col"
                  class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                  No. HP</th>
                <th scope="col"
                  class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                  Alamat</th>
                <th scope="col"
                  class="px-6 py-3.5 text-left text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                  Tanggal Lahir</th>
                <th scope="col"
                  class="px-6 py-3.5 text-right text-xs font-semibold text-gray-500 dark:text-gray-300 uppercase tracking-wider">
                  Aksi</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-100 dark:divide-gray-700">
              <tr v-for="user in users" :key="user.id"
                class="hover:bg-indigo-50/40 dark:hover:bg-gray-700/50 transition-colors">
                <td class="px-6 py-4">
                  <div class="flex items-center gap-3">
                    <div :class="avatarClass(user.username)"
                      class="w-10 h-10 rounded-full flex items-center justify-center text-white text-sm font-bold shrink-0">
                      {{ getInitials(user.full_name || user.username) }}
                    </div>
                    <div class="min-w-0">
                      <p class="text-sm font-semibold text-gray-800 dark:text-white truncate">{{ user.full_name }}</p>
                      <p class="text-xs text-gray-500 dark:text-gray-400 truncate">@{{ user.username }}</p>
                    </div>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600 dark:text-gray-300">{{ user.phone || '-'
                  }}</td>
                <td class="px-6 py-4 text-sm text-gray-600 dark:text-gray-300 max-w-[240px] truncate"
                  :title="user.address">{{ user.address || '-' }}</td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600 dark:text-gray-300">{{
                  formatTanggalIndo(user.dob) }}</td>
                <td class="px-6 py-4 whitespace-nowrap text-right">
                  <div class="inline-flex items-center gap-2">
                    <button @click="openEditModal(user)"
                      class="inline-flex items-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                      <Icon icon="heroicons:pencil-square" class="w-4 h-4" />
                      Edit
                    </button>
                    <button @click="confirmDelete(user)"
                      class="inline-flex items-center gap-1.5 text-red-600 dark:text-red-400 bg-red-50 dark:bg-red-900/30 hover:bg-red-100 dark:hover:bg-red-900/60 px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors">
                      <Icon icon="heroicons:trash" class="w-4 h-4" />
                      Hapus
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
        <div v-for="user in users" :key="user.id"
          class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 p-4">
          <div class="flex items-center gap-3 mb-3">
            <div :class="avatarClass(user.username)"
              class="w-11 h-11 rounded-full flex items-center justify-center text-white text-sm font-bold shrink-0">
              {{ getInitials(user.full_name || user.username) }}
            </div>
            <div class="min-w-0 flex-1">
              <p class="text-sm font-semibold text-gray-800 dark:text-white truncate">{{ user.full_name }}</p>
              <p class="text-xs text-gray-500 dark:text-gray-400 truncate">@{{ user.username }}</p>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-x-4 gap-y-2 text-sm mb-3">
            <div class="flex items-center gap-2 text-gray-600 dark:text-gray-300 min-w-0">
              <Icon icon="heroicons:phone" class="w-4 h-4 text-gray-400 shrink-0" />
              <span class="truncate">{{ user.phone || '-' }}</span>
            </div>
            <div class="flex items-center gap-2 text-gray-600 dark:text-gray-300 min-w-0">
              <Icon icon="heroicons:cake" class="w-4 h-4 text-gray-400 shrink-0" />
              <span class="truncate">{{ formatTanggalIndo(user.dob) }}</span>
            </div>
            <div class="flex items-center gap-2 text-gray-600 dark:text-gray-300 col-span-2 min-w-0">
              <Icon icon="heroicons:map-pin" class="w-4 h-4 text-gray-400 shrink-0" />
              <span class="truncate">{{ user.address || '-' }}</span>
            </div>
          </div>
          <div class="flex gap-2 pt-3 border-t border-gray-100 dark:border-gray-700">
            <button @click="openEditModal(user)"
              class="flex-1 inline-flex items-center justify-center gap-1.5 text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 hover:bg-indigo-100 dark:hover:bg-indigo-900/60 px-3 py-2 rounded-xl text-sm font-semibold transition-colors">
              <Icon icon="heroicons:pencil-square" class="w-4 h-4" />
              Edit
            </button>
            <button @click="confirmDelete(user)"
              class="flex-1 inline-flex items-center justify-center gap-1.5 text-red-600 dark:text-red-400 bg-red-50 dark:bg-red-900/30 hover:bg-red-100 dark:hover:bg-red-900/60 px-3 py-2 rounded-xl text-sm font-semibold transition-colors">
              <Icon icon="heroicons:trash" class="w-4 h-4" />
              Hapus
            </button>
          </div>
        </div>
      </div>
    </template>

    <!-- Add user modal -->
    <div v-if="showAddModal"
      class="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
      @click.self="closeModal">
      <div
        class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-2xl max-h-[92vh] flex flex-col overflow-hidden">
        <div class="px-5 sm:px-6 py-4 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-3">
            <div
              class="w-9 h-9 rounded-xl bg-indigo-50 dark:bg-indigo-900/40 flex items-center justify-center text-indigo-600 dark:text-indigo-400">
              <Icon icon="heroicons:user-plus" class="w-5 h-5" />
            </div>
            <h2 class="text-lg font-bold text-gray-800 dark:text-white">Tambah Pengguna Baru</h2>
          </div>
          <button @click="closeModal"
            class="p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors">
            <Icon icon="heroicons:x-mark" class="w-5 h-5" />
          </button>
        </div>
        <div class="p-5 sm:p-6 overflow-y-auto">
          <div v-if="addErrorMessage"
            class="mb-4 p-3 text-sm text-red-700 dark:text-red-300 bg-red-50 dark:bg-red-900/30 border border-red-200 dark:border-red-800 rounded-xl flex items-center gap-2">
            <Icon icon="heroicons:exclamation-triangle-solid" class="w-4 h-4 shrink-0 text-red-500" />
            {{ addErrorMessage }}
          </div>
          <form @submit.prevent="submitAddUser" class="space-y-4">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Username</label>
                <input v-model="newUser.username" type="text" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Kata Sandi</label>
                <input v-model="newUser.password" type="password" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Nama Lengkap</label>
                <input v-model="newUser.full_name" type="text" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Tanggal Lahir</label>
                <input v-model="newUser.dob" type="date" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">No. HP</label>
              <input v-model="newUser.phone" type="tel"
                class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Alamat</label>
              <textarea v-model="newUser.address" required rows="2"
                class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all resize-none"></textarea>
            </div>
            <div
              class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3 pt-4 border-t border-gray-100 dark:border-gray-700 mt-6">
              <button type="button" @click="closeModal"
                class="px-4 py-2.5 text-sm font-medium text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-xl transition-colors">Batal</button>
              <button type="submit" :disabled="isAdding"
                class="inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2.5 rounded-xl text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
                <Icon v-if="isAdding" icon="heroicons:arrow-path" class="w-4 h-4 animate-spin" />
                {{ isAdding ? 'Menyimpan...' : 'Simpan Pengguna' }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Edit user modal -->
    <div v-if="showEditModal"
      class="fixed inset-0 z-50 flex items-end sm:items-center justify-center bg-black/50 backdrop-blur-sm sm:px-4"
      @click.self="closeEditModal">
      <div
        class="bg-white dark:bg-gray-800 rounded-t-2xl sm:rounded-2xl shadow-xl w-full max-w-2xl max-h-[92vh] flex flex-col overflow-hidden">
        <div class="px-5 sm:px-6 py-4 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-3">
            <div
              class="w-9 h-9 rounded-xl bg-indigo-50 dark:bg-indigo-900/40 flex items-center justify-center text-indigo-600 dark:text-indigo-400">
              <Icon icon="heroicons:pencil-square" class="w-5 h-5" />
            </div>
            <h2 class="text-lg font-bold text-gray-800 dark:text-white">Edit Pengguna</h2>
          </div>
          <button @click="closeEditModal"
            class="p-1.5 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-100 dark:hover:text-gray-300 dark:hover:bg-gray-700 transition-colors">
            <Icon icon="heroicons:x-mark" class="w-5 h-5" />
          </button>
        </div>
        <div class="p-5 sm:p-6 overflow-y-auto">
          <div v-if="editErrorMessage"
            class="mb-4 p-3 text-sm text-red-700 dark:text-red-300 bg-red-50 dark:bg-red-900/30 border border-red-200 dark:border-red-800 rounded-xl flex items-center gap-2">
            <Icon icon="heroicons:exclamation-triangle-solid" class="w-4 h-4 shrink-0 text-red-500" />
            {{ editErrorMessage }}
          </div>
          <form @submit.prevent="submitEditUser" class="space-y-4">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Username
                  <span class="text-xs text-gray-400">(tidak dapat diubah)</span></label>
                <input :value="editUser.username" type="text" disabled
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-100 dark:bg-gray-700 dark:border-gray-600 text-gray-500 dark:text-gray-400 cursor-not-allowed" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Kata Sandi Baru
                  <span class="text-xs text-gray-400">(opsional)</span></label>
                <input v-model="editUser.password" type="password" placeholder="Kosongkan jika tidak diubah"
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Nama Lengkap</label>
                <input v-model="editUser.full_name" type="text" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Tanggal Lahir</label>
                <input v-model="editUser.dob" type="date" required
                  class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
              </div>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">No. HP</label>
              <input v-model="editUser.phone" type="tel"
                class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all" />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1.5">Alamat</label>
              <textarea v-model="editUser.address" required rows="2"
                class="w-full px-3.5 py-2.5 text-sm border border-gray-200 rounded-xl bg-gray-50 dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500 focus:bg-white dark:focus:bg-gray-900 transition-all resize-none"></textarea>
            </div>
            <div
              class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3 pt-4 border-t border-gray-100 dark:border-gray-700 mt-6">
              <button type="button" @click="closeEditModal"
                class="px-4 py-2.5 text-sm font-medium text-gray-600 bg-gray-100 hover:bg-gray-200 dark:text-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600 rounded-xl transition-colors">Batal</button>
              <button type="submit" :disabled="isEditing"
                class="inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2.5 rounded-xl text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
                <Icon v-if="isEditing" icon="heroicons:arrow-path" class="w-4 h-4 animate-spin" />
                {{ isEditing ? 'Menyimpan...' : 'Perbarui Pengguna' }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, reactive, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import { Icon } from '@iconify/vue';

const token = localStorage.getItem('token');
const baseUrl = 'https://alentest.my.id/tabungan';
const router = useRouter()

const users = ref([]);
const isLoading = ref(true);
const errorMessage = ref('');
const globalSuccessMessage = ref('');

const showAddModal = ref(false);
const isAdding = ref(false);
const addErrorMessage = ref('');
const newUser = reactive({
  username: '',
  password: '',
  full_name: '',
  dob: '',
  address: '',
  phone: ''
});

const showEditModal = ref(false);
const isEditing = ref(false);
const editErrorMessage = ref('');
const editUser = reactive({
  id: '',
  username: '',
  password: '',
  full_name: '',
  dob: '',
  address: '',
  phone: ''
});

const searchQuery = reactive({
  username: '',
  dob: ''
});

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

const formatToDDMMYYYY = (dateString) => {
  if (!dateString) return '';
  const [year, month, day] = dateString.split('-');
  return `${day}-${month}-${year}`;
};

const formatToYYYYMMDD = (dateString) => {
  if (!dateString) return '';

  if (typeof dateString === 'string') {
    const normalizedDate = dateString.includes('T') ? dateString.split('T')[0] : dateString;
    if (/^\d{4}-\d{2}-\d{2}$/.test(normalizedDate)) return normalizedDate;
  }

  const date = new Date(dateString);
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

const formatTanggalIndo = (dateString) => {
  if (!dateString) return '-';
  const normalizedDate = formatToYYYYMMDD(dateString);

  if (!normalizedDate) return '-';

  const [year, month, day] = normalizedDate.split('-').map(Number);
  const date = new Date(year, month - 1, day);
  return date.toLocaleDateString('id-ID', {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  });
};

const fetchUsers = async () => {
  isLoading.value = true;
  errorMessage.value = '';
  try {
    const response = await fetch(`${baseUrl}/api/users`, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    const data = await response.json();
    users.value = data.data || data;
  } catch (error) {
    errorMessage.value = `Gagal memuat data: ${error.message}`;
    router.push({
      name: "Login"
    })
  } finally {
    isLoading.value = false;
  }
};

const handleSearch = async () => {
  if (!searchQuery.username && !searchQuery.dob) {
    return fetchUsers();
  }

  isLoading.value = true;
  errorMessage.value = '';

  try {
    const url = new URL(`${baseUrl}/api/users/search`);
    if (searchQuery.username) url.searchParams.append('username', searchQuery.username);
    if (searchQuery.dob) {
      const formattedDob = formatToDDMMYYYY(searchQuery.dob);
      url.searchParams.append('dob', formattedDob);
    }

    const response = await fetch(url.toString(), {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });

    if (response.status === 404) {
      users.value = [];
      return;
    }

    const data = await response.json();
    if (!response.ok) throw new Error(data.message || 'Terjadi kesalahan');

    if (Array.isArray(data)) {
      users.value = data;
    } else if (data && Array.isArray(data.data)) {
      users.value = data.data;
    } else {
      users.value = [data];
    }

  } catch (error) {
    errorMessage.value = error.message;
  } finally {
    isLoading.value = false;
  }
};

let searchTimeout;
watch(searchQuery, () => {
  clearTimeout(searchTimeout);
  searchTimeout = setTimeout(() => {
    handleSearch();
  }, 500);
}, { deep: true });

const clearSearch = () => {
  searchQuery.username = '';
  searchQuery.dob = '';
};

const closeModal = () => {
  showAddModal.value = false;
  addErrorMessage.value = '';
  Object.assign(newUser, { username: '', password: '', full_name: '', dob: '', address: '', phone: '' });
};

const submitAddUser = async () => {
  isAdding.value = true;
  addErrorMessage.value = '';

  try {
    const response = await fetch(`${baseUrl}/api/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify(newUser)
    });

    const data = await response.json();
    if (!response.ok) throw new Error(data.message || `Gagal menambahkan pengguna.`);

    globalSuccessMessage.value = `Pengguna ${newUser.full_name} berhasil ditambahkan!`;
    closeModal();
    fetchUsers();
    clearSearch();
  } catch (error) {
    addErrorMessage.value = error.message;
  } finally {
    isAdding.value = false;
  }
};

const openEditModal = (user) => {
  editUser.id = user.id;
  editUser.username = user.username;
  editUser.full_name = user.full_name;
  editUser.dob = formatToYYYYMMDD(user.dob);
  editUser.address = user.address;
  editUser.phone = user.phone || '';
  editUser.password = '';
  editErrorMessage.value = '';
  showEditModal.value = true;
};

const closeEditModal = () => {
  showEditModal.value = false;
  editErrorMessage.value = '';
};

const submitEditUser = async () => {
  isEditing.value = true;
  editErrorMessage.value = '';

  const payload = {
    full_name: editUser.full_name,
    dob: editUser.dob,
    address: editUser.address,
    phone: editUser.phone
  };

  if (editUser.password) {
    payload.password = editUser.password;
  }

  try {
    const response = await fetch(`${baseUrl}/api/users/${editUser.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify(payload)
    });

    const data = await response.json();
    if (!response.ok) throw new Error(data.message || `Gagal memperbarui pengguna.`);

    globalSuccessMessage.value = `Data ${editUser.full_name} berhasil diperbarui!`;
    closeEditModal();
    fetchUsers();
  } catch (error) {
    editErrorMessage.value = error.message;
  } finally {
    isEditing.value = false;
  }
};

const confirmDelete = async (user) => {
  const isConfirmed = window.confirm(`Apakah Anda yakin ingin menghapus pengguna "${user.username}"? Tindakan ini tidak dapat dibatalkan.`);

  if (!isConfirmed) return;

  try {
    const response = await fetch(`${baseUrl}/api/users/${user.id}`, {
      method: 'DELETE',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });

    const data = await response.json();
    if (!response.ok) throw new Error(data.message || `Gagal menghapus pengguna.`);

    globalSuccessMessage.value = `Pengguna ${user.username} berhasil dihapus!`;
    fetchUsers();
  } catch (error) {
    alert(`Error: ${error.message}`);
  }
};

onMounted(() => {
  fetchUsers();
});
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
