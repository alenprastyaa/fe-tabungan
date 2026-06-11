<template>
  <div class="p-4 sm:p-6 lg:p-8  mx-auto">
    <div class="mb-6">
      <h1 class="text-2xl font-bold text-gray-900 dark:text-white">Setting Laporan Email</h1>
      <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">
        Backup data pelanggan, tabungan, dan transaksi akan dikirim sebagai file Excel.
      </p>
    </div>

    <div v-if="message"
      :class="messageIsError ? 'bg-red-50 text-red-700 border-red-200 dark:bg-red-900/30 dark:text-red-300 dark:border-red-800' : 'bg-emerald-50 text-emerald-700 border-emerald-200 dark:bg-emerald-900/30 dark:text-emerald-300 dark:border-emerald-800'"
      class="mb-5 border px-4 py-3 rounded-lg text-sm">
      {{ message }}
    </div>

    <div class="bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-lg overflow-hidden">
      <div class="p-5 sm:p-6 border-b border-gray-100 dark:border-gray-700">
        <div class="flex items-center justify-between gap-4">
          <div>
            <h2 class="font-semibold text-gray-900 dark:text-white">Jadwal Otomatis</h2>
            <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">Aktifkan pengiriman harian pada jam yang dipilih.
            </p>
          </div>
          <label class="inline-flex items-center cursor-pointer">
            <input v-model="form.enabled" type="checkbox" class="sr-only peer">
            <span
              class="relative w-11 h-6 bg-gray-200 rounded-full peer peer-checked:bg-indigo-600 after:content-[''] after:absolute after:top-0.5 after:left-0.5 after:bg-white after:w-5 after:h-5 after:rounded-full after:transition-all peer-checked:after:translate-x-5"></span>
          </label>
        </div>
      </div>

      <div class="p-5 sm:p-6 space-y-6">
        <div>
          <label class="block text-sm font-semibold text-gray-700 dark:text-gray-200 mb-2">Email Penerima</label>
          <textarea v-model="recipientsText" rows="5"
            class="w-full px-3 py-2.5 text-sm border border-gray-200 rounded-lg dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500"
            placeholder="admin@email.com&#10;owner@email.com"></textarea>
          <p class="text-xs text-gray-500 dark:text-gray-400 mt-2">Tulis satu email per baris. Laporan akan dikirim ke
            semua alamat ini.</p>
        </div>

        <div>
          <label class="block text-sm font-semibold text-gray-700 dark:text-gray-200 mb-2">Jam Kirim</label>
          <div class="flex flex-col sm:flex-row gap-3">
            <input v-model="newTime" type="time"
              class="px-3 py-2.5 text-sm border border-gray-200 rounded-lg dark:bg-gray-900 dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-indigo-500/40 focus:border-indigo-500" />
            <button @click="addTime" type="button"
              class="inline-flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2.5 rounded-lg text-sm font-semibold transition-colors">
              <Icon icon="heroicons:plus" class="w-4 h-4" />
              Tambah Jam
            </button>
          </div>

          <div v-if="form.schedule_times.length" class="flex flex-wrap gap-2 mt-3">
            <span v-for="time in form.schedule_times" :key="time"
              class="inline-flex items-center gap-2 px-3 py-1.5 bg-gray-100 dark:bg-gray-700 text-gray-800 dark:text-gray-100 rounded-lg text-sm">
              <Icon icon="heroicons:clock" class="w-4 h-4" />
              {{ time }}
              <button @click="removeTime(time)" type="button" class="text-gray-400 hover:text-red-500">
                <Icon icon="heroicons:x-mark" class="w-4 h-4" />
              </button>
            </span>
          </div>
          <p v-else class="text-xs text-gray-500 dark:text-gray-400 mt-2">Belum ada jam kirim.</p>
        </div>
      </div>

      <div
        class="p-5 sm:p-6 bg-gray-50 dark:bg-gray-800 border-t border-gray-100 dark:border-gray-700 flex flex-col sm:flex-row gap-3 sm:justify-end">
        <button @click="sendNow" :disabled="loading || sending"
          class="inline-flex items-center justify-center gap-2 px-4 py-2.5 rounded-lg border border-gray-200 dark:border-gray-600 text-gray-700 dark:text-gray-100 hover:bg-white dark:hover:bg-gray-700 disabled:opacity-50 text-sm font-semibold transition-colors">
          <Icon :icon="sending ? 'heroicons:arrow-path' : 'heroicons:paper-airplane'"
            :class="sending ? 'animate-spin' : ''" class="w-4 h-4" />
          {{ sending ? 'Mengirim...' : 'Kirim Sekarang' }}
        </button>
        <button @click="saveSettings" :disabled="loading || saving"
          class="inline-flex items-center justify-center gap-2 bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2.5 rounded-lg text-sm font-semibold disabled:opacity-50 transition-colors">
          <Icon :icon="saving ? 'heroicons:arrow-path' : 'heroicons:check'" :class="saving ? 'animate-spin' : ''"
            class="w-4 h-4" />
          {{ saving ? 'Menyimpan...' : 'Simpan Setting' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';
import { Icon } from '@iconify/vue';

const baseUrl = process.env.VUE_APP_API_BASE_URL || 'https://alentest.my.id/tabungan';
const token = localStorage.getItem('token');
const router = useRouter();

const loading = ref(false);
const saving = ref(false);
const sending = ref(false);
const message = ref('');
const messageIsError = ref(false);
const recipientsText = ref('');
const newTime = ref('');

const form = reactive({
  recipients: [],
  schedule_times: [],
  enabled: false
});

const showMessage = (text, isError = false) => {
  message.value = text;
  messageIsError.value = isError;
  setTimeout(() => {
    message.value = '';
  }, 5000);
};

const parseRecipients = () => recipientsText.value
  .split(/\n|,/)
  .map((email) => email.trim())
  .filter(Boolean);

const handleUnauthorized = (response) => {
  if (response.status === 401 || response.status === 403) {
    router.push({ name: 'Login' });
    return true;
  }
  return false;
};

const parseJsonResponse = async (response) => {
  const contentType = response.headers.get('content-type') || '';

  if (!contentType.includes('application/json')) {
    throw new Error('Endpoint laporan belum tersedia di backend. Restart/update backend lalu coba lagi.');
  }

  return response.json();
};

const fetchSettings = async () => {
  loading.value = true;
  try {
    const response = await fetch(`${baseUrl}/api/report-settings`, {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (handleUnauthorized(response)) return;

    const data = await parseJsonResponse(response);
    if (!response.ok) throw new Error(data.message || 'Gagal mengambil setting laporan');

    form.recipients = data.recipients || [];
    form.schedule_times = data.schedule_times || [];
    form.enabled = Boolean(data.enabled);
    recipientsText.value = form.recipients.join('\n');
  } catch (error) {
    showMessage(error.message, true);
  } finally {
    loading.value = false;
  }
};

const addTime = () => {
  if (!newTime.value) return;
  if (!form.schedule_times.includes(newTime.value)) {
    form.schedule_times = [...form.schedule_times, newTime.value].sort();
  }
  newTime.value = '';
};

const removeTime = (time) => {
  form.schedule_times = form.schedule_times.filter((item) => item !== time);
};

const saveSettings = async () => {
  saving.value = true;
  try {
    const response = await fetch(`${baseUrl}/api/report-settings`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`
      },
      body: JSON.stringify({
        recipients: parseRecipients(),
        schedule_times: form.schedule_times,
        enabled: form.enabled
      })
    });

    if (handleUnauthorized(response)) return;

    const data = await parseJsonResponse(response);
    if (!response.ok) throw new Error(data.message || 'Gagal menyimpan setting laporan');

    form.recipients = data.recipients || [];
    form.schedule_times = data.schedule_times || [];
    form.enabled = Boolean(data.enabled);
    recipientsText.value = form.recipients.join('\n');
    showMessage('Setting laporan berhasil disimpan');
  } catch (error) {
    showMessage(error.message, true);
  } finally {
    saving.value = false;
  }
};

const sendNow = async () => {
  sending.value = true;
  try {
    const response = await fetch(`${baseUrl}/api/report-settings/send-now`, {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (handleUnauthorized(response)) return;

    const data = await parseJsonResponse(response);
    if (!response.ok) throw new Error(data.message || 'Gagal mengirim laporan');

    showMessage(`Laporan berhasil dikirim ke ${data.recipients?.join(', ') || 'email terdaftar'}`);
  } catch (error) {
    showMessage(error.message, true);
  } finally {
    sending.value = false;
  }
};

onMounted(fetchSettings);
</script>
