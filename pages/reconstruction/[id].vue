<script setup lang="ts">
const { t } = useI18n()
const { params } = useRoute()
const bus = useEventBus('reconstruction')

const user = useUser()

const { data, error } = await useApi<Competition>(`/reconstruction/${params.id}`)
if (!data.value || error.value) {
  throw createError({
    statusCode: 404,
  })
}
const competition = reactive<Competition>(data.value)
const submissions = reactive<Record<number, Submission[]>>({})
const mySubmissions = computed(() => {
  const ret: Record<number, Submission[]> = {}
  for (const { id } of competition.scrambles)
    ret[id] = submissions[id]?.filter(submission => submission.user.id === user.id) ?? []
  return ret
})

await fetchSubmissions()
async function fetchSubmissions() {
  const { data, refresh } = await useApi<Record<number, Submission[]>>(`/reconstruction/${params.id}/submissions`, {
    immediate: false,
  })
  await refresh()
  if (data.value)
    Object.assign(submissions, data.value)
}

useSeoMeta({
  title: t('reconstruction.title'),
})
useIntervalFn(fetchSubmissions, 5000)
bus.on(fetchSubmissions)
</script>

<template>
  <div>
    <h1 class="font-bold text-xl md:text-3xl my-2">
      {{ competition.name }}
    </h1>
    <div class="flex gap-2 mt-4">
      <p class="font-bold text-lg">
        {{ $t('reconstruction.date') }}:
        {{ $dayjs(competition.startTime).format('YYYY-MM-DD') }}
      </p>
      <a
        v-if="competition.wcaCompetitionId"
        :href="competition.wcaCompetitionId"
        target="_blank"
        class="text-blue-500"
      >
        <WcaLogo class="w-6" />
      </a>
    </div>
    <Tabs>
      <Tab
        v-for="scramble in competition.scrambles"
        :key="scramble.id"
        :name="$t('reconstruction.scramble', { round: scramble.round, group: scramble.group, number: scramble.number })"
        :hash="`scramble-${scramble.number}`"
      >
        <Sequence :sequence="scramble.scramble" :source="scramble.scramble" />
        <CubeExpanded :moves="scramble.scramble" />
        <ReconstructionForm
          :scramble="scramble"
          :competition="competition"
          :submissions="mySubmissions[scramble.id]"
          @submitted="fetchSubmissions"
        />
        <h2 class="text-lg font-semibold my-2">
          {{ $t('reconstruction.solutions') }}
        </h2>
        <Submissions :submissions="submissions[scramble.id]" />
      </Tab>
    </Tabs>
  </div>
</template>
