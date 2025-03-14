<template>
  <UiCard>
    <UiCardTitle
      :left="$t('storage-usage')"
      :right="$t('top-#', { n: N_ITEMS })"
    />
    <UsageBar
      :data="srStore.isReady ? data.result : undefined"
      :nItems="N_ITEMS"
    >
      <template #footer v-if="showFooter">
        <div class="footer-card">
          <p>{{ $t("total-used") }}:</p>
          <div class="footer-value">
            <p>{{ percentUsed }}%</p>
            <p>
              {{ formatSize(data.usedSize) }}
            </p>
          </div>
        </div>
        <div class="footer-card">
          <p>{{ $t("total-free") }}:</p>
          <div class="footer-value">
            <p>{{ percentFree }}%</p>
            <p>
              {{ formatSize(data.maxSize) }}
            </p>
          </div>
        </div>
      </template>
    </UsageBar>
  </UiCard>
</template>

<script lang="ts" setup>
import UiCardTitle from "@/components/ui/UiCardTitle.vue";
import { computed } from "vue";
import UsageBar from "@/components/UsageBar.vue";
import UiCard from "@/components/ui/UiCard.vue";
import { formatSize, percent } from "@/libs/utils";
import { useSrStore } from "@/stores/storage.store";
import { N_ITEMS } from "@/views/pool/PoolDashboardView.vue";

const srStore = useSrStore();

const percentUsed = computed(() =>
  percent(data.value.usedSize, data.value.maxSize, 1)
);

const percentFree = computed(() =>
  percent(data.value.maxSize - data.value.usedSize, data.value.maxSize, 1)
);

const showFooter = computed(() => !isNaN(percentUsed.value));

const data = computed<{
  result: { id: string; label: string; value: number }[];
  maxSize: number;
  usedSize: number;
}>(() => {
  const result: { id: string; label: string; value: number }[] = [];
  let maxSize = 0;
  let usedSize = 0;

  srStore.allRecords.forEach(
    ({ name_label, physical_size, physical_utilisation, uuid }) => {
      if (physical_size < 0 || physical_utilisation < 0) {
        return;
      }

      maxSize += physical_size;
      usedSize += physical_utilisation;

      const percent = (physical_utilisation / physical_size) * 100;

      if (isNaN(percent)) {
        return;
      }

      result.push({
        id: uuid,
        label: name_label,
        value: percent,
      });
    }
  );
  return { result, maxSize, usedSize };
});
</script>

<style lang="postcss" scoped>
.footer-card {
  color: var(--color-blue-scale-200);
  display: flex;
  text-transform: uppercase;
}

.footer-card p {
  font-weight: 700;
}

.footer-value {
  display: flex;
  flex-direction: column;
  text-align: right;
}
</style>
