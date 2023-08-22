<template>
  <p-layout-default>
    <template #header>
      <PageHeading :crumbs="crumbs" />
    </template>

    <template v-if="taskRunsSubscription.executed && taskRuns.length">
      <TaskRunList :task-runs="taskRuns" />
      <p-pager v-model:page="page" :pages="pages" />
    </template>
    <template v-else-if="taskRunsSubscription.executed">
      <p>No task runs</p>
    </template>
    <template v-else>
      <p>Loading...</p>
    </template>
  </p-layout-default>
</template>

<script lang="ts" setup>
  import { BreadCrumbs } from '@prefecthq/prefect-design'
  import { useFilterPagination, useWorkspaceApi, TaskRunList, TaskRunsFilter, PageHeading } from '@prefecthq/prefect-ui-library'
  import { useSubscription } from '@prefecthq/vue-compositions'
  import { computed, ref } from 'vue'

  const api = useWorkspaceApi()

  const page = ref(1)
  const { limit, offset } = useFilterPagination(page, 10)

  const filter = computed<[TaskRunsFilter]>(() => {
    return [
      {
        limit: limit.value,
        offset: offset.value,
      },
    ]
  })

  const taskRunsSubscription = useSubscription(api.taskRuns.getTaskRuns, filter)
  const taskRunsCountSubscription = useSubscription(api.taskRuns.getTaskRunsCount, filter)

  const taskRuns = computed(() => taskRunsSubscription.response ?? [])
  const taskRunsCount = computed(() => taskRunsCountSubscription.response)

  const pages = computed(() => Math.ceil((taskRunsCount.value ?? 0) / limit.value))

  const crumbs: BreadCrumbs = [
    {
      text: 'Task Runs',
    },
  ]
</script>