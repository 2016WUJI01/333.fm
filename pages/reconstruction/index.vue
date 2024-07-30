<script setup lang="ts">
const { t } = useI18n()
useSeoMeta({
  title: t('reconstruction.title'),
})
const route = useRoute()

const { data } = await useApi<Pagination<Reconstruction>>('/reconstruction', {
  params: {
    page: route.query.page,
  },
})
if (!data.value) {
  throw createError({
    statusCode: 500,
  })
}
const reconstructions = ref<Reconstruction[]>(data.value.items)
</script>

<template>
  <div>
    <h1 class="font-bold text-lg md:text-3xl my-2">
      {{ $t('reconstruction.title') }}
    </h1>
    <p class="mb-2">
      {{ $t('reconstruction.description') }}
    </p>
    <div
      class="grid grid-cols-auto md:grid-cols-[max-content_max-content_max-content_1fr] gap-x-2 gap-y-1 md:gap-y-2
            mb-2"
    >
      <div class="font-bold">
        {{ $t('reconstruction.date') }}
      </div>
      <div class="font-bold">
        {{ $t('reconstruction.name') }}
      </div>
      <div class="font-bold" />
      <div class="col-span-3 md:col-span-1" />
      <template v-for="{ id, date, name } in reconstructions " :key="id">
        <div class="flex items-center">
          {{ $dayjs(date).format('YYYY-MM-DD') }}
        </div>
        <div class="flex items-center flex-wrap">
          {{ name }}
        </div>
        <div class="text-indigo-500 font-semibold flex items-center">
          <NuxtLink :to="`/reconstruction/${id}`" class="bg-indigo-500 text-white px-3 py-2 text-lg">
            {{ $t('reconstruction.join') }}
          </NuxtLink>
        </div>
      </template>
    </div>
  </div>
</template>
