<script setup>
// Mengambil data dari Python (pastikan port-nya benar)
const { data, pending, error } = await useFetch('http://127.0.0.1:8000/api/data')
</script>

<template>
  <UContainer class="py-10">
    <UCard>
      <template #header>
        <h3 class="text-xl font-bold">Koneksi Nuxt + Python</h3>
      </template>
      <div v-if="pending">
        Sedang memuat data dari Python...
      </div>
      
      <div v-else-if="data">
        <p class="text-green-500 font-medium">{{ data.message }}</p>
        <UBadge color="primary">{{ data.status }}</UBadge>
      </div>
      <div v-else class="text-red-500">
        Gagal terhubung ke server Python. Pastikan port 8000 sudah nyala.
      </div>
    </UCard>
  </UContainer>
</template>