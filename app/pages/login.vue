<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '#ui/types'

const formRef = ref()
const router = useRouter()
const toast = useToast()

const state = reactive({
  email: '',
  password: '',
  remember: false
})

const schema = z.object({
  email: z.email('Format email salah'),
  password: z.string('Password is required').min(8, 'Must be at least 8 characters')
})

type Schema = z.output<typeof schema>

async function onSubmit(event: FormSubmitEvent<Schema>) {
  formRef.value.clear()

  try {
    const response: any = await $fetch('http://localhost:5050/login', {
      method: 'POST',
      body: state
    })

    if (response.success) {
      toast.add({ title: 'Success', color: 'success' })
      router.push('/home')
    }

  } catch (err: any) {
    console.log("ERROR DARI PYTHON:", err.data)
    const errorData = err.data

    if (errorData && errorData.field) {
      formRef.value.setErrors([{
        path: errorData.field,  
        message: errorData.message 
      }])
    } else {
      toast.add({ title: 'Gagal', description: 'Terjadi kesalahan sistem', color: 'error' })
    }
  }
}
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen gap-4 p-4 bg-gray-50 dark:bg-gray-900">
    <UCard class="w-full max-w-md">
      <template #header>
        <h1 class="text-xl font-bold">Login</h1>
      </template>
      <UForm ref="formRef" :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        
        <UFormField label="Email" name="email">
          <UInput v-model="state.email" icon="i-heroicons-envelope" />
        </UFormField>
        <UFormField label="Password" name="password">
          <UInput v-model="state.password" type="password" icon="i-heroicons-lock-closed" />
        </UFormField>
        <UButton type="submit" block>Masuk</UButton>
        <div class="flex items-center justify-center gap-2 mt-4 text-sm">
          <span class="text-gray-500">Belum punya akun?</span>
          <UButton to="/register" variant="link" color="primary" :padded="false">
            Daftar
          </UButton>
        </div>
      </UForm>
    </UCard>
  </div>
</template>