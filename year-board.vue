<template>
  <BaseCard download class="my-4 litigation-card" title="本年案件看板" style="border-radius: 16px">
    <template #headerRight>
      <BaseDownloadButton class="year-download-btn" btnText="下载图表" :buttonProps="{type: 'outline'}"></BaseDownloadButton>
    </template>
    <div class="grid grid-cols-2 gap-4">
      <div v-for="caseItem in yearCaseList" :key="caseItem.id" class="relative pt-1">
        <img class="absolute top-0 left-0 z-50" width="44" :src="IconPlaintiff" alt="">
        <div class="year-card" :class="[`year-${caseItem.id}`]">
          <p class="text-gradient mb-3 text-lg font-semibold relative z-10">{{ caseItem.title }}</p>
          <ul class="flex items-center year-box-list relative z-10">
            <li
              v-for="(item, index) in caseItem.data"
              :key="item.title"
              :class="[`year-box-${index}`, 'cursor-pointer']"
              @click="turnPage(item.config, '', 'Board')"
            >
              <p class="year-box-title mb-2">{{ item.title }}</p>
              <p class="year-box-num">
                <span class="mr-2"><span class="text-gradient">{{ item.data?.num }}</span> 件</span>
                <span :class="item?.yoyNum >= 0 ? 'text-primary': 'color-fall'" v-if="item.title === '本年新发'">
                  <span :class="{'text-[#969AA3]': item?.yoyNum === 0}">同比 {{ Math.abs(item?.yoyNum) }}%</span>
                  <span v-if="Math.abs(item?.yoyNum) !== 0">
                    <icon-arrow-rise v-if="item?.yoyNum >= 0" class="text-primary" />
                    <icon-arrow-fall v-else class="color-fall" />
                  </span>
                </span>
              </p>
              <p class="year-box-amount">
                <span class="text-[#656A73] mr-2">{{ item.formatAmount }}</span>
                <span :class="item?.yoyAmount >= 0 ? 'text-primary': 'color-fall'" v-if="item.title === '本年新发'">
                  <span :class="{'text-[#969AA3]': item?.yoyAmount === 0}">同比 {{ Math.abs(item?.yoyAmount) }}%</span>
                  <span v-if="Math.abs(item?.yoyNum) !== 0">
                    <icon-arrow-rise v-if="item?.yoyAmount >= 0" class="text-primary" />
                    <icon-arrow-fall v-else class="color-fall" />
                  </span>
                </span>
              </p>
            </li>
          </ul>
          <img class="bg-circle absolute" :src="caseItem.id === 'defendant' ? yearBgCircle1 : yearBgCircle2" alt="">
          <img class="bg-rect absolute" :src="caseItem.id === 'defendant' ? yearBgRect1 : yearBgRect2" alt="">
        </div>
      </div>
    </div>
  </BaseCard>
</template>

<script lang="ts" setup>
import { computed } from 'vue';
import { currencyFormat } from '@/utils/currency';
import { turnPage } from '@/utils/process';
import yearBgCircle1 from '@/assets/images/year_bgcircle_1.png';
import yearBgCircle2 from '@/assets/images/year_bgcircle_2.png';
import yearBgRect1 from '@/assets/images/year_bgrect_1.png';
import yearBgRect2 from '@/assets/images/year_bgrect_2.png';
import IconPlaintiff from '@/assets/images/icon-plaintiff.png';
import BaseDownloadButton from './base-download-button';
import BaseCard from './base-card';
import { useGetBoardStatistics } from '../api/dashboard';
import { BoardType, RecentDataType } from '../enums/board';
import useBoardStoreFilters from '../store';

const filterState = useBoardStoreFilters();
const { data: data30, isFetching: fetching1 } = useGetBoardStatistics(
  BoardType.YEAR_BOARD_DEFENDANT,
  {
    recentDataType: BoardType.YEAR_BOARD_DEFENDANT
  }
);
const { data: data31, isFetching: fetching2 } = useGetBoardStatistics(
  BoardType.YEAR_BOARD_PLAINTIFF,
  {
    recentDataType: BoardType.YEAR_BOARD_PLAINTIFF
  }
);

// 获取同比数据
const getYoyData = (item: any) => {
  let yoyNum = 0;
  let yoyAmount = 0;
  if (item.title === '本年新发') {
    if (item.data?.oldNum) {
      if (item.data?.oldNum > 0) {
        yoyNum = Math.floor((item.data.num - item.data.oldNum) / item.data.oldNum * 100)
      }
    }
    if (item.data?.oldAmount) {
      if (item.data?.oldAmount > 0) {
        yoyAmount = Math.floor((item.data.amount - item.data.oldAmount) / item.data.oldAmount * 100)
      }
    }
  }
  return { yoyNum, yoyAmount }
}

// 被诉
const defendantData = computed(() => {
  if (!data30.value) return [];
  return data30.value.map(item => {
    const [currency, suffix] = currencyFormat(item.data?.amount || 0, 2);
    const { yoyNum, yoyAmount } = getYoyData(item);
    return {
      ...item,
      formatAmount: `${currency} ${suffix}元`,
      yoyNum,
      yoyAmount
    }
  })
})

// 主诉
const plaintiffData = computed(() => {
  if (!data31.value) return [];
  return data31.value.map(item => {
    const [currency, suffix] = currencyFormat(item.data?.amount || 0, 2);
    const { yoyNum, yoyAmount } = getYoyData(item);
    return {
      ...item,
      formatAmount: `${currency} ${suffix}元`,
      yoyNum,
      yoyAmount
    }
  })
})

const yearCaseList = computed(() => {
  return [
    {
      id: 'defendant',
      title: '被诉',
      data: defendantData.value,
    },
    {
      id: 'plaintiff',
      title: '主诉',
      data: plaintiffData.value,
    }
  ]
})
</script>

<style lang="scss" scoped>
:deep(.year-download-btn) {
  border: none !important;
  padding-right: 0 !important;
  >.arco-btn-icon {
    margin-right: 4px;
  }
}
.year-card {
  border-radius: 16px;
  padding: 16px 4px 16px 24px;
  position: relative;
  overflow: hidden;
  &.year-defendant {
    background: linear-gradient(177.75deg, #FEEFE6 10.57%, rgba(255, 250, 245, 0.62) 95.29%);

    .text-gradient {
      color: #F47C32;
    }
  }

  &.year-plaintiff {
    background: linear-gradient(177.67deg, #F3F5FF 11.55%, rgba(248, 248, 255, 0.33) 93.45%);
    .text-gradient {
      color: #515A86;
    }
    .year-box-list {
      li.year-box-0 {
        &::after {
          background-color: rgba(82, 91, 135, .1);
        }
      }
    }
  }

  .year-box-list {
    li {
      .year-box-title {
        font-weight: 500;
        font-size: 14px;
        color: #11192B;
      }

      .year-box-num,
      .year-box-amount {
        font-weight: 400;
        font-size: 12px;
        color: #969AA3;
        display: flex;
        align-items: center;
        .text-gradient {
          font-family: 'DIN Alternate';
          font-weight: 700;
          font-size: 20px;
        }
      }
      &:hover {
        .year-box-title {
          color: #F47C32;
        }
        .text-gradient {
          color: #F47C32;
        }
      }
    }

    li:not(.year-box-0) {
      flex: 1;
    }

    li.year-box-0 {
      width: 30%;
      position: relative;
      margin-right: 32px;
      padding-right: 10px;
      &::after {
        position: absolute;
        top: 14px;
        right: 0;
        width: 1px;
        height: 50px;
        content: '';
        background-color: rgba(244, 124, 50, .1);
      }
    }
  }
  .bg-circle {
    left: -70px;
    top: 20px;
    z-index: 0;
  }
  .bg-rect {
    right: 0;
    top: -120px;
    z-index: 0;
  }
  .color-fall {
    color: #07C575;
  }
}
</style>
