<script setup lang="ts">
import { getPaginationRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import { h, resolveComponent } from 'vue'

definePageMeta({
  title: 'Список продуктів'
})

useHead({
  title: 'Список продуктів'
})

const productsTable = useTemplateRef('productsTable')
const URatingBadge = resolveComponent('UBadge')

interface IProductData {
  products: IProduct[]
}

interface IProduct {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
  images: string[]
}

const { data: productList, status: loadingStatus } = await useFetch('https://dummyjson.com/products', {
  key: 'products-data',
  transform: (response: IProductData): IProduct[] => {
    return response.products.map(item => ({
      ...item,
      brand: item.brand || 'Без бренду',
      rating: item.rating
    }))
  },
  lazy: true
})

const tableColumns: TableColumn<IProduct>[] = [
  {
    accessorKey: 'title',
    header: 'Назва',
    cell: ({ row }: { row: { original: IProduct } }) => h('div', row.original.title)
  },

  {
    accessorKey: 'description',
    header: 'Опис',
    cell: ({ row }: { row: { original: IProduct } }) => h('div', {
      class: 'w-[200px] whitespace-normal break-words line-clamp-3'
    }, row.original.description)
  },
  {
    accessorKey: 'price',
    header: 'Ціна',
    cell: ({ row }: { row: { original: IProduct } }) => {
      return h('div', { class: 'text-right' }, `${row.original.price}$`)
    }
  },
  {
    accessorKey: 'rating',
    header: 'Оцінка',
    cell: ({ row }: { row: { original: IProduct } }) => {
      const ratingValue = row.original.rating
      const badgeColor = ratingValue < 4.5 ? 'error' : 'success'
      return h(URatingBadge, {
        variant: 'subtle',
        color: badgeColor
      }, () => ratingValue.toFixed(1))
    }
  },
  {
    accessorKey: 'brand',
    header: 'Бренд',
    cell: ({ row }: { row: { original: IProduct } }) => h('div', row.original.brand)
  },
  {
    accessorKey: 'category',
    header: 'Категорія',
    cell: ({ row }: { row: { original: IProduct } }) => h('div', row.original.category)
  },
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    cell: ({ row }: { row: { original: IProduct } }) => {
      return h('img', {
        src: row.original.thumbnail,
        width: 100,
        height: 100,
        style: 'object-fit: cover; border-radius: 4px'
      })
    }
  }
]

const searchFilter = ref('')
const paginationSettings = ref({
  pageIndex: 0,
  pageSize: 7
})

const handleSearchInput = () => {
  paginationSettings.value.pageIndex = 0
}

const handlePageChange = (page: number) => {
  productsTable.value?.tableApi?.setPageIndex(page - 1)
}

watch(searchFilter, handleSearchInput)
</script>

<template>
  <div class="w-full max-w-screen-lg mx-auto space-y-4 pb-4">
    <div class="flex px-4 py-3.5 border-b border-gray-200">
      <UInput
          v-model="searchFilter"
          class="max-w-sm"
          placeholder="Пошук товарів..."
          icon="i-heroicons-magnifying-glass"
      />
    </div>

    <UTable
        ref="productsTable"
        v-model:global-filter="searchFilter"
        v-model:pagination="paginationSettings"
        :data="productList || []"
        :columns="tableColumns"
        :pagination-options="{
        getPaginationRowModel: getPaginationRowModel()
      }"
        class="w-full text-left"
    />

    <div class="flex justify-center border-t border-gray-200 pt-4">
      <UPagination
          :default-page="(productsTable?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="productsTable?.tableApi?.getState().pagination.pageSize"
          :total="productsTable?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="handlePageChange"
      />
    </div>
  </div>
</template>