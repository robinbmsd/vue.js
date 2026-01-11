<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent, AuthFormField } from '@nuxt/ui'

definePageMeta({
  layout: false
})

const fields: AuthFormField[] = [{
  name: 'email',
  type: 'email',
  label: 'Email',
  placeholder: 'Enter your email',
  required: true
}, {
  name: 'password',
  label: 'Password',
  type: 'password',
  placeholder: 'Enter your password',
  required: true
}, {
  name: 'remember',
  label: 'Remember me',
  type: 'checkbox'
}]

const schema = z.object({
  email: z.email('Invalid email'),
  password: z.string('Password is required').min(8, 'Must be at least 8 characters')
})

type Schema = z.output<typeof schema>

const router = useRouter()
function onSubmit(payload: FormSubmitEvent<Schema>) {
  console.log('Submitted', payload)
  router.push('/home')
}
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen gap-4 p-4">
    <UPageCard class="w-full max-w-md">
      <div class="flex justify-end mb-4">
        <UColorModeSwitch />
      </div>

      <UAuthForm
        :schema="schema"
        title="Verify"
        description="Your code was sent to you via email"
        icon="i-lucide-key"
        :fields="fields"
        @submit="onSubmit"
      />
    </UPageCard>
  </div>
</template>
