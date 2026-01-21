<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui'

const route = useRoute()
const router = useRouter()
const toast = useToast()

const items: NavigationMenuItem[] = [
  {
    label: 'Home',
    icon: 'i-lucide-house',
    to: 'home',
    active: false
  },
  {
    label: 'Transaction History',
    icon: 'i-lucide-clipboard-clock',
    to: 'transaction_history',
    active: false
  }
]

const pageTitle = computed((): string => {
  if (route.meta.title) {
    return route.meta.title as string
  }
  
  const name = route.name || 'Dashboard'
  return name.toString().replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase())
})

const pageIcon = computed(() => {
  const activeItem = items.find(item => item.to === route.name)
  return activeItem ? activeItem.icon : 'i-lucide-box' 
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
    console.error('Logout error:', error)
    if (import.meta.client) {
      localStorage.removeItem('user_email')
    }
    router.push('/login')
  }
}
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
        <UDashboardNavbar>
          
          <template #title>
            <div class="flex items-center gap-2">
              <UIcon 
                :name="pageIcon" 
                class="w-5 h-5 text-primary-500" 
              />
    
              <span class="text-gray-900 dark:text-white font-semibold truncate">
                {{ pageTitle }}
              </span>
            </div>
      </template>

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