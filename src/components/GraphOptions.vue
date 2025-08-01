<template>
  <Collapsible
    v-model:open="isOpen"
    class="space-y-2 bg-card shadow-xl shadow-amber-50 rounded-md mb-2"
  >
    <CollapsibleTrigger as-child>
      <Button variant="ghost" size="sm" class="font-bold w-full p-0 m-0">
        Graph Control
        <ChevronsUpDown class="h-4 w-4" />
        <span class="sr-only">Toggle</span>
      </Button>
    </CollapsibleTrigger>
    <CollapsibleContent class="p-4 space-y-2">
      <div class="flex gap-2 flex-wrap">
        <div class="control-group">
          <Button id="layout-toggle-v2" @click="toggleLayout">
            {{ layout === 'force' ? 'Switch to Circle Pack' : 'Switch to Force Directed' }}
          </Button>
        </div>
        <div class="control-group">
          <Button id="refresh-btn-v2" @click="refreshGraph">Refresh Graph</Button>
        </div>

        <div class="control-group">
          <Button id="reset-btn-v2" @click="resetGraph">Reset Graph</Button>
        </div>

        <div class="control-group">
          <Button id="load-more-btn-v2" :disabled="!loadMoreBtn.status" @click="loadMoreData">
            {{ loadMoreBtn.text }}
          </Button>
        </div>
      </div>

      <div class="flex gap-2 flex-wrap">
        <Select
          v-model="nodeSelect.selected as string"
          @update:modelValue="focusOnNode(String($event))"
        >
          <SelectTrigger class="w-[180px]">
            <SelectValue placeholder="Select a node" />
          </SelectTrigger>
          <SelectContent>
            <SelectGroup>
              <SelectItem v-for="item in nodeSelect.options" :key="item.value" :value="item.value">
                {{ item.label }}
              </SelectItem>
            </SelectGroup>
          </SelectContent>
        </Select>

        <div class="control-group w-">
          <label for="threshold-slider-v2">
            Label Threshold:
            <span>{{ labelThreshold }}</span>
          </label>

          <Slider
            id="threshold-slider-v2"
            v-model="labelThreshold"
            @update:model-value="updateThreshold"
            :min="0"
            :max="10"
            :step="0.1"
          />
        </div>
      </div>
    </CollapsibleContent>
  </Collapsible>
</template>

<script setup lang="ts">
import { ref, inject, computed } from 'vue'
import Collapsible from './ui/collapsible/Collapsible.vue'
import CollapsibleTrigger from './ui/collapsible/CollapsibleTrigger.vue'
import Button from './ui/button/Button.vue'
import { ChevronsUpDown } from 'lucide-vue-next'
import CollapsibleContent from './ui/collapsible/CollapsibleContent.vue'
import Slider from './ui/slider/Slider.vue'
import Select from './ui/select/Select.vue'
import SelectTrigger from './ui/select/SelectTrigger.vue'
import SelectValue from './ui/select/SelectValue.vue'
import SelectContent from './ui/select/SelectContent.vue'
import SelectGroup from './ui/select/SelectGroup.vue'
import SelectItem from './ui/select/SelectItem.vue'
import type { GraphContext } from '../types/graph-context'

// Inject the graph context from parent component
const graphContext = inject<GraphContext>('graphContext')

if (!graphContext) {
  throw new Error('GraphOptions must be used within a component that provides graphContext')
}

const {
  graph,
  dataManager,
  layout,
  labelThreshold,
  loadMoreBtn,
  nodeSelect,
  fetchLoading,
  updateLoadingIndicator,
} = graphContext

// Local state
const isOpen = ref(true)

// Methods
function toggleLayout() {
  layout.value = layout.value === 'force' ? 'circlepack' : 'force'
  graph.value?.setLayout(layout.value as 'force' | 'circlepack')
}

function refreshGraph() {
  graph.value?.refresh()
}

function resetGraph() {
  graph.value?.resetGraph()
}

async function loadMoreData() {
  fetchLoading.value = true
  loadMoreBtn.value.status = false
  try {
    const newData = await dataManager.value?.fetchNextPage()
    if (newData && graph.value) {
      graph.value.addData(newData)
      nodeSelect.value.options.push(
        ...graph.value.getNodesData().map((node) => ({
          label: node.label || String(node.id),
          value: node.id.toString(),
        })),
      )
    }
    console.log('nodes', graph.value?.getDataSize().nodes)
    console.log('links', graph.value?.getDataSize().links)
  } catch (error) {
    console.error('Error fetching data:', error)
  }
  fetchLoading.value = false
  updateLoadingIndicator()
}

function updateThreshold() {
  graph.value?.setOptions({
    labelThreshold: labelThreshold.value[0],
  })
}

function focusOnNode(nodeId: string) {
  if (!graph.value) return

  if (nodeId && graph.value.hasNode(nodeId)) {
    graph.value.focusPosition({ id: nodeId })

    // Optional: Show feedback
    const node = graph.value.getNodeById(nodeId)
    if (node) {
      console.log(`Focused on node: ${node.label || nodeId}`)
    }
  } else {
    console.error('Node not found or invalid ID provided.')
  }
}
</script>
