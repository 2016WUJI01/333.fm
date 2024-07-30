<script setup lang='ts'>
const props = defineProps<{
  scramble: Scramble
  competition: Competition
  submissions: Submission[]
}>()
const emit = defineEmits<{
  submitted: []
}>()
const submissionsMap = computed(() => {
  const ret: Record<number, Submission> = {}
  for (const submission of props.submissions) ret[submission.mode] = submission
  return ret
})

const form = reactive({
  mode: CompetitionMode.MYSELF,
  wcaId: '',
  solution: '',
  comment: '',
})

const localForm = useLocalStorage<Record<number, Record<number, { solution: string, comment: string }>>>(`form.weekly.${props.competition.alias}`, {})
onMounted(() => {
  const localValue = submissionsMap.value[form.mode] || localForm.value[props.scramble.number]?.[form.mode]
  if (localValue) {
    form.solution = localValue.solution
    form.comment = localValue.comment
  }
})
watch(form, (state) => {
  localForm.value = {
    ...localForm.value,
    [props.scramble.number]: {
      ...localForm.value[props.scramble.number],
      [form.mode]: {
        solution: state.solution,
        comment: state.comment,
      },
    },
  }
})

watch(() => form.mode, (mode) => {
  const localValue = submissionsMap.value[mode] || localForm.value[props.scramble.number]?.[mode]
  if (localValue) {
    form.solution = localValue.solution
    form.comment = localValue.comment
  }
  else {
    form.solution = ''
    form.comment = ''
  }
})

const { moves, isSolved } = useComputedState({
  scramble: {
    id: props.scramble.id,
    number: props.scramble.number,
    scramble: props.scramble.scramble,
    createdAt: props.scramble.createdAt,
    updatedAt: props.scramble.updatedAt,
  },
}, form)

const solutionState = computed<boolean | null>(() => {
  if (form.mode === CompetitionMode.OTHERS && form.wcaId === '')
    return null

  if (form.solution.length === 0 || !isSolved.value)
    return null

  return moves.value !== DNF
})

const formState = computed<boolean>(() => {
  return solutionState.value !== null
})

const loading = ref(false)

async function submit() {
  loading.value = true
  try {
    const { data, refresh } = await useApiPost<Submission>(`/reconstruction/${props.competition.id}`, {
      body: {
        scrambleId: props.scramble.id,
        ...form,
      },
      immediate: false,
    })
    await refresh()
    if (data.value)
      emit('submitted')
  }
  catch (e: any) {
    if (e.response && e.response.data && e.response.data.message)
      alert(e.response.data.message)

    else alert(e.message)
  }
  finally {
    loading.value = false
  }
}

function reset() {
  if (!submissionsMap.value[form.mode])
    form.solution = ''
  form.comment = ''
}
</script>

<template>
  <div class="mt-6">
    <form class="relative" @submit="submit" @reset="reset">
      <FormSignInRequired />
      <FormInput
        v-model="form.mode"
        type="radio"
        :label="$t('reconstruction.mode.label')"
        :state="null"
        :attrs="{ require: true }"
        :options="[
          { label: $t('reconstruction.myself.label'), description: $t('reconstruction.myself.description'), value: CompetitionMode.MYSELF },
        ]"
      >
        <template v-if="form.mode === CompetitionMode.OTHERS" #default>
          <FormInput
            v-model="form.wcaId"
            type="text"
            :label="$t('reconstruction.others.wcaId')"
            :state="null"
            :attrs="{ required: true }"
          />
        </template>
      </FormInput>
      <FormInput
        v-model="form.solution"
        type="textarea"
        :rows="4"
        :label="$t('reconstruction.solution.label')"
        :state="solutionState"
        :attrs="{ required: true }"
      >
        <template #description>
          <div v-if="moves !== DNF" class="text-green-500 text-bold">
            {{ $t('common.moves', { moves }) }}
          </div>
          <div v-else class="text-red-500 text-bold">
            DNF
          </div>
          <I18nT keypath="if.scramble.description" tag="p" scope="global">
            <template #notation>
              <Notation />
            </template>
          </I18nT>
        </template>
      </FormInput>
      <FormInput
        v-model="form.comment"
        type="textarea"
        :rows="4"
        :label="$t('reconstruction.comment.label')"
        :state="null"
        class="mt-4"
      >
        <template #description>
          <p class="py-1" v-html="$t('reconstruction.comment.description')" />
        </template>
      </FormInput>
      <div class="mt-4">
        <button
          class="px-2 py-1 text-white bg-blue-500 focus:outline-none"
          :class="{ 'bg-opacity-50 cursor-not-allowed': !formState }"
          :disabled="!formState"
          @click.prevent="submit"
        >
          <Spinner v-if="loading" class="w-4 h-4 text-white border-[3px]" />
          <template v-else>
            {{ $t('form.submit') }}
          </template>
        </button>
        <button class="px-2 py-1 text-white bg-gray-500 focus:outline-none ml-2" @click.prevent="reset">
          {{ $t('form.reset') }}
        </button>
      </div>
    </form>
  </div>
</template>
