<script setup lang="ts">
import { getPaginationRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import type { Column } from '@tanstack/vue-table'
import { h, resolveComponent } from 'vue'

const UButton = resolveComponent('UButton')
const UDropdownMenu = resolveComponent('UDropdownMenu')

useHead({
  title: 'Список продуктів'
})

const table = useTemplateRef('table')

type ProductResponse = {
  products: Product[]
}

type Product = {
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}
const { data } = await useFetch('https://dummyjson.com/products', {
  key: 'table-products',
  transform: (response: ProductResponse): Product[] => {
    return response.products.map(product => ({
      ...product,
      brand: product.brand || 'No Brand',
    }))
  },
  lazy: true
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: ({ column }) => getHeader(column, 'Title')
  },
  {
    accessorKey: 'description',
    header: 'Description',
    cell: ({ row }) => h('div', {
      style: 'width: 400px; white-space: normal; word-break: break-word;'
    }, row.original.description)
  },
  {
    accessorKey: 'price',
    header: ({ column }) => getHeader(column, 'Price')
  },
  {
    accessorKey: 'rating',
    header: ({ column }) => getHeader(column, 'Rating'),
    cell: ({ row }) => {
      const rating = row.original.rating
      const color = rating < 4.5 ? 'text-red-500' : 'text-green-500'

      return h('span', { class: color }, rating.toFixed(1))
    }
  },
  {
    accessorKey: 'brand',
    header: ({ column }) => getHeader(column, 'Brand')
  },
  {
    accessorKey: 'category',
    header: ({ column }) => getHeader(column, 'Category')
  },
  {
    accessorKey: 'thumbnail',
    header: 'Photo',
    cell: ({ row }) => {
      const url = row.original.thumbnail
      return h('img', {
        src: url,
        width: 100,
        height: 100,
        style: 'object-fit: cover; border-radius: 8px'
      })
    }
  }
]


const globalFilter = ref('')

const pagination = ref({
  pageIndex: 0,
  pageSize: 5

})
const handleFilterChange = () => {
  pagination.value.pageIndex = 0
}

function getHeader(column: Column<Product>, label: string) {
  const isSorted = column.getIsSorted()

  return h(
      UDropdownMenu,
      {
        content: {
          align: 'start'
        },
        'aria-label': 'Actions dropdown',
        items: [
          {
            label: 'Asc',
            type: 'checkbox',
            icon: 'i-lucide-arrow-up-narrow-wide',
            checked: isSorted === 'asc',
            onSelect: () => {
              if (isSorted === 'asc') {
                column.clearSorting()
              } else {
                column.toggleSorting(false)
              }
            }
          },
          {
            label: 'Desc',
            icon: 'i-lucide-arrow-down-wide-narrow',
            type: 'checkbox',
            checked: isSorted === 'desc',
            onSelect: () => {
              if (isSorted === 'desc') {
                column.clearSorting()
              } else {
                column.toggleSorting(true)
              }
            }
          }
        ]
      },
      () =>
          h(UButton, {
            color: 'neutral',
            variant: 'ghost',
            label,
            icon: isSorted
                ? isSorted === 'asc'
                    ? 'i-lucide-arrow-up-narrow-wide'
                    : 'i-lucide-arrow-down-wide-narrow'
                : 'i-lucide-arrow-up-down',
            class: '-mx-2.5 data-[state=open]:bg-elevated',
            'aria-label': `Sort by ${isSorted === 'asc' ? 'descending' : 'ascending'}`
          })
  )
}

watch(globalFilter, handleFilterChange)
</script>

<template>
  <div class="w-full px-15 mx-auto space-y-4 pb-4">
    <div class="flex justify-center px-4 py-3.5">
      <UInput v-model="globalFilter" class="w-full max-w-sm" placeholder="Filter..." clearable />
    </div>

    <UTable
        ref="table"
        v-model:global-filter="globalFilter"
        v-model:pagination="pagination"
        :data="data || []"
        :columns="columns"
        :pagination-options="{
    getPaginationRowModel: getPaginationRowModel()
  }"
        class="w-full text-left flex-1 "
    />

    <div class="flex justify-center border-t border-(--ui-border) pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>