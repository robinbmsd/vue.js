<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui'

const route = useRoute()
const router = useRouter()
const toast = useToast()

const pageTitle = computed(() => {
  const name = route.name || 'Dashboard'
  return name.toString().charAt(0).toUpperCase() + name.toString().slice(1)
})

const onLogout = async () => {
  try {
    await $fetch('http://localhost:8000/logout', {
      method: 'POST'
    })

    if (import.meta.client) {
      localStorage.removeItem('user_email')
    }

    toast.add({
      title: 'Keluar',
      description: 'Anda telah berhasil logout',
      color: 'success'
    })

    router.push('/login')
  } catch (error) {
    // Tambahkan blok ini untuk memperbaiki parsing error
    console.error('Logout error:', error)
    // Tetap hapus localstorage & pindah halaman jika server gagal merespon
    if (import.meta.client) {
      localStorage.removeItem('user_email')
    }
    router.push('/login')
  }
}

const items: NavigationMenuItem[] = [{
  label: 'Home',
  icon: 'i-lucide-house',
  to: 'home',
  active: false
}, {
  label: 'Database',
  icon: 'i-lucide-database',
  to: 'database',
  active: false

}]
</script>

<template>
  <UDashboardGroup>
    <UDashboardSidebar
      close
      :toggle="{
        color: 'primary',
        variant: 'subtle',
        class: 'rounded-full'
      }"
    >
      <template #header>
        <Logo class="h-5 w-auto" />
      </template>

      <UNavigationMenu
        :items="items"
        orientation="vertical"
      />

      <template #footer>
        <UButton
          icon="i-lucide-log-out"
          label="Logout"
          color="error"
          variant="ghost"
          block
          @click="onLogout"
        />
      </template>
    </UDashboardSidebar>

    <UDashboardPanel>
      <template #header>
        <UDashboardNavbar :title="pageTitle">
          <template #right>
            <UColorModeSwitch />

            <UButton
              icon="i-lucide-log-out"
              color="neutral"
              variant="ghost"
              class="ml-2"
              @click="onLogout"
            />
          </template>
        </UDashboardNavbar>
      </template>

      <template #body>
        <slot />
      </template>
    </UDashboardPanel>
  </UDashboardGroup>
</template>
