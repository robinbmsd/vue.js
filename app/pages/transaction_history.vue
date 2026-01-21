<script setup lang="ts">
import type { Column } from '@tanstack/vue-table'
import { h, resolveComponent, ref, onMounted, watch } from 'vue'
import type { TableColumn } from '@nuxt/ui'

const UBadge = resolveComponent('UBadge')
const UButton = resolveComponent('UButton')
const UDropdownMenu = resolveComponent('UDropdownMenu')
const UTooltip = resolveComponent('UTooltip') 
const table = useTemplateRef('table')

definePageMeta({
  layout: 'dashboard',
  title: 'Transaction History'
})

type UserData = {
  number: string
  client_id: string
  trx_id: string
  phone: string
  date: string
  status: 'success' | 'failed' | 'pending'
  serial_number: string
  error: string      
  
  b_id: string 
  
  amount: number
  respon_time: number
}

const data = ref<UserData[]>([])
const isLoading = ref(false)
const globalFilter = ref('')
const selectedDate = ref('')
const sorting = ref([{ id: 'number', desc: true }]) 
const pagination = ref({ pageIndex: 0, pageSize: 10 })

async function fetchData() {
  isLoading.value = true
  try {
    let url = 'http://127.0.0.1:8000/users'
    if (selectedDate.value) {
      url += `?date=${selectedDate.value}`
    }

    const response = await fetch(url)
    const result = await response.json()
    data.value = result
  } catch (error) {
    console.error("Gagal konek API:", error)
  } finally {
    isLoading.value = false
  }
}

watch(selectedDate, () => {
  fetchData()
})

onMounted(() => {
  fetchData()
})

function getHeader(column: Column<UserData>, label: string) {
  const isSorted = column.getIsSorted()
  return h(UDropdownMenu, {
      'content': { align: 'start' },
      'items': [
        { label: 'Asc', type: 'checkbox', checked: isSorted === 'asc', onSelect: () => column.toggleSorting(false) },
        { label: 'Desc', type: 'checkbox', checked: isSorted === 'desc', onSelect: () => column.toggleSorting(true) }
      ]
    },
    () => h(UButton, { 
        color: 'neutral', 
        variant: 'ghost', 
        label, 
        icon: isSorted ? (isSorted === 'asc' ? 'i-lucide-arrow-up' : 'i-lucide-arrow-down') : 'i-lucide-arrow-up-down' 
    })
  )
}

const columns: TableColumn<UserData>[] = [
  {
    accessorKey: 'number',
    header: ({ column }) => getHeader(column, 'No.'),
    cell: ({ row }) => `#${row.getValue('number')}`
  },
  {
    accessorKey: 'client_id',
    header: ({ column }) => getHeader(column, 'Client ID'),
    cell: ({ row }) => h('div', { class: 'font-semibold text-gray-900 dark:text-white' }, row.getValue('client_id') as string)
  },
  {
    accessorKey: 'trx_id',
    header: ({ column }) => getHeader(column, 'Trx ID'),
    cell: ({ row }) => {
      const fullId = row.getValue('trx_id') as string
      if (!fullId) return '-'
      const shortId = fullId.substring(0, 8) + '...'
      return h(UTooltip, { text: fullId }, () => 
        h('span', { 
          class: 'font-mono text-xs cursor-help border-b border-dotted border-gray-400 hover:text-primary-500 transition-colors' 
        }, shortId)
      )
    }
  },
  {
    accessorKey: 'phone',
    header: ({ column }) => getHeader(column, 'Phone No.'),
    cell: ({ row }) => {
      const phone = row.getValue('phone') as string
      return phone ? h('div', {class: 'font-mono text-nowrap'}, phone) : '-'
    }
  },
  {
    accessorKey: 'date',
    header: ({ column }) => getHeader(column, 'Date'),
    cell: ({ row }) => {
      const dateStr = row.getValue('date') as string
      if (!dateStr) return '-' 
      return new Date(dateStr).toLocaleString('id-ID', {
        day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit', hour12: false
      })
    }
  },
  {
    accessorKey: 'status',
    header: ({ column }) => getHeader(column, 'Status'),
    cell: ({ row }) => {
      const rawStatus = row.getValue('status') as string
      const statusKey = rawStatus ? rawStatus.toLowerCase() : ''
      const color = { success: 'success', failed: 'error', pending: 'neutral' }[statusKey] || 'gray' 
      return h(UBadge, { class: 'capitalize', variant: 'subtle', color }, () => rawStatus)
    }
  },
  {
    accessorKey: 'serial_number',
    header: ({ column }) => getHeader(column, 'Serial Number'),
    cell: ({ row }) => {
      const sn = row.getValue('serial_number') as string
      return sn ? h('div', {class: 'font-mono text-nowrap'}, sn) : '-'
    }
  },
  { accessorKey: 'error', header: ({ column }) => getHeader(column, 'Error Desc.') },
  
  // KOLOM B_ID
  { 
    accessorKey: 'b_id', 
    header: ({ column }) => getHeader(column, 'BID'), 
    cell: ({ row }) => h('div', {class: 'font-mono'}, row.getValue('b_id') as string) 
  },
  
  {
    accessorKey: 'amount',
    header: ({ column }) => h('div', { class: 'text-right' }, getHeader(column, 'Amount (IDR)')),
    cell: ({ row }) => {
      const amount = Number(row.getValue('amount'))
      const formatted = new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(amount)
      return h('div', { class: 'text-right font-medium' }, formatted)
    }
  },
  {
    accessorKey: 'respon_time',
    header: ({ column }) => h('div', { class: 'text-right' }, getHeader(column, 'Tsel Resp. Time')),
    cell: ({ row }) => {
      const val = Number(row.getValue('respon_time'))
      if (isNaN(val)) return '-'
      let colorClass = 'text-gray-500' 
      if (val > 20000) colorClass = 'text-red-500'
      else if (val > 5000) colorClass = 'text-yellow-500'
      return h('div', { class: `font-mono text-right ${colorClass}` }, `${new Intl.NumberFormat('id-ID').format(val)} ms`)
    }
  },
]
</script>

<template>
  <div class="flex flex-col flex-1 w-full">
    <div class="flex px-4 py-3.5 border-b border-accented justify-between items-center gap-4">
      
      <div class="flex items-center gap-2">
        <UInput 
          v-model="selectedDate" 
          type="date" 
          placeholder="Pilih Tanggal" 
          icon="i-lucide-calendar"
          class="w-48"
        />
        <UButton 
          v-if="selectedDate"
          icon="i-lucide-x"
          variant="ghost" 
          color="gray"
          size="xs"
          @click="selectedDate = ''"
          tooltip="Hapus Filter"
        />
      </div>

      <div class="flex items-center gap-3">
        <UInput 
          v-model="globalFilter" 
          class="max-w-sm" 
          placeholder="Search..." 
          icon="i-lucide-search"
        />
        <UButton 
          icon="i-lucide-refresh-cw" 
          variant="ghost" 
          color="gray" 
          :loading="isLoading"
          @click="fetchData"
        >
          Refresh Data
        </UButton>
      </div>

    </div>

    <UTable 
      ref="table" 
      v-model:pagination="pagination" 
      v-model:global-filter="globalFilter" 
      v-model:sorting="sorting" 
      :data="data" 
      :columns="columns" 
      :loading="isLoading"
      class="flex-1" 
    />
    
    <div class="flex justify-end border-t border-default pt-4 px-4">
      <UPagination
        :page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
        :items-per-page="table?.tableApi?.getState().pagination.pageSize"
        :total="table?.tableApi?.getFilteredRowModel().rows.length"
        @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>