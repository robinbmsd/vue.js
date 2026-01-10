<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '#ui/types'

const formRef = ref()
const router = useRouter()
const toast = useToast()

const state = reactive({
  email: '',
  password: ''
})

const schema = z.object({
  email: z.email('Format email tidak valid'),
  password: z.string().min(8, 'Password minimal 8 karakter')
})

type Schema = z.output<typeof schema>

async function onSubmit(event: FormSubmitEvent<Schema>) {
  formRef.value.clear()

  try {
    const response: any = await $fetch('http://localhost:6060/signup', {
      method: 'POST',
      body: state
    })

    if (response.success) {
      toast.add({ 
        title: 'Berhasil', 
        description: 'Akun dibuat! Silakan login.', 
        color: 'success' 
      })
      router.push('/login')
    }

  } catch (err: any) {
    const errorData = err.data

    if (errorData && errorData.field) {
      formRef.value.setErrors([{
        path: errorData.field,
        message: errorData.message
      }])
    } else {
      toast.add({ 
        title: 'Gagal', 
        description: 'Terjadi kesalahan sistem', 
        color: 'error' 
      })
    }
  }
}
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen gap-4 p-4 bg-gray-50 dark:bg-gray-900">
    <UCard class="w-full max-w-md">
      <template #header>
        <h1 class="text-xl font-bold">Daftar Akun Baru</h1>
        <p class="text-sm text-gray-500">Isi data untuk mendaftar</p>
      </template>

      <UForm ref="formRef" :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        
        <UFormField label="Email" name="email">
          <UInput v-model="state.email" icon="i-heroicons-envelope" placeholder="email@contoh.com" />
        </UFormField>

        <UFormField label="Password" name="password">
          <UInput v-model="state.password" type="password" icon="i-heroicons-lock-closed" placeholder="Minimal 8 karakter" />
        </UFormField>

        <UButton type="submit" block color="primary">
          Daftar Sekarang
        </UButton>

        <div class="flex items-center justify-center gap-2 mt-4 text-sm">
          <span class="text-gray-500">Sudah punya akun?</span>
          <UButton to="/login" variant="link" color="neutral" :padded="false">
            Login di sini
          </UButton>
        </div>

      </UForm>
    </UCard>
  </div>
</template>