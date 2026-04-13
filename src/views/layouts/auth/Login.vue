<template>
  <div class="w-full h-screen">
    <div class="flex shadow rounded-md h-screen">
      <div class="bg-white dark:bg-gray-900 w-full">
        <form @submit.prevent="handleLogin">
          <div class="form-body lg:max-w-xl mx-auto lg:p-20 p-8 lg:mt-20 mt-5 space-y-8">
            <div class="form-head cursor-pointer" @click="router.push('/')">
              <img src="../../../assets/logo/logo.svg" alt="" class="w-10" />
            </div>
            <div class="space-y-3">
              <h2 class="dark:text-white font-semibold text-gray-800 text-4xl">
                Selamat datang diTabungan Lebaran<span class="text-primary">.</span>
              </h2>
              <p class="dark:text-gray-400 text-gray-700">
                Masukan username dan password anda
              </p>
            </div>

            <div v-if="errorMessage" class="p-3 text-sm text-red-500 bg-red-100 rounded-md">
              {{ errorMessage }}
            </div>

            <div class="space-y-5">
              <div class="relative z-0 w-full mb-6 group">
                <input type="text" name="floating_username" id="floating_username" v-model="form.username"
                  class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none dark:text-white dark:border-gray-600 dark:focus:border-primary focus:outline-none peer"
                  placeholder=" " required />
                <label for="floating_username"
                  class="peer-focus:font-medium absolute text-sm text-gray-500 dark:text-gray-400 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:left-0 peer-focus:text-primary peer-focus:dark:text-primary peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">
                  Username
                </label>
              </div>
              <div class="relative z-0 w-full mb-6 group mt-6">
                <input type="password" name="floating_password" id="floating_password" v-model="form.password"
                  class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none dark:text-white dark:border-gray-600 dark:focus:border-primary focus:outline-none peer"
                  placeholder=" " required />
                <label for="floating_password"
                  class="peer-focus:font-medium absolute text-sm text-gray-500 dark:text-gray-400 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:left-0 peer-focus:text-primary peer-focus:dark:text-primary peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">
                  Password
                </label>
              </div>
            </div>

            <div class="flex justify-between">


            </div>

            <button type="submit" :disabled="isLoading"
              class="text-white bg-primary hover:bg-primary/80 p-3 w-full rounded-md disabled:opacity-50 disabled:cursor-not-allowed">
              {{ isLoading ? 'Logging in...' : 'Login, to continue' }}
            </button>
          </div>
        </form>
      </div>
      <div class="bg-wave dark:bg-gray-900 bg-white w-2/5 lg:block hidden"></div>
    </div>
  </div>
</template>
<script setup>
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const form = reactive({
  username: '',
  password: '',
  remember: false
});

const isLoading = ref(false);
const errorMessage = ref('');

const handleLogin = async () => {
  isLoading.value = true;
  errorMessage.value = '';

  try {
    const baseUrl = 'http://localhost:2100';

    const response = await fetch(`${baseUrl}/api/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify({
        username: form.username,
        password: form.password
      })
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.message || 'Gagal login. Periksa kembali username dan password Anda.');
    }
    localStorage.setItem('token', data.token);
    localStorage.setItem('role', data.role)
    const userData = {
      id: data.id,
      username: data.username,
      full_name: data.full_name,
      role: data.role,
      dob: data.dob
    };
    localStorage.setItem('user', JSON.stringify(userData));
    router.push('/');

  } catch (error) {
    errorMessage.value = error.message;
  } finally {
    isLoading.value = false;
  }
};
</script>

<style>
.bg-wave {
  background: radial-gradient(circle at top left,
      transparent 25%,
      #4f46e5 25.5%,
      #4f46e5 36%,
      transparent 37%,
      transparent 100%),
    radial-gradient(circle at bottom right,
      transparent 34%,
      #4f46e5 34.5%,
      #4f46e5 45.5%,
      transparent 46%,
      transparent 100%);
  background-size: 6em 6em;
  opacity: 1;
}
</style>