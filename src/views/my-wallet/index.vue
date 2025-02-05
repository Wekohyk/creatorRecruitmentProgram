<script setup lang="ts">
import { onMounted, ref } from 'vue';
import PageLayout from '@/components/PageLayout.vue';
import { getAuthor, getWallet } from '@/api';
import { Wallet } from '@/types/user';
import hintCard from './hintCard.vue';
import individualTaxInstructionPopup from './individualTaxInstructionPopup.vue';
import keyboard from './keyboard.vue';

const myWallet = ref<Wallet>();
const authorLevel = ref();

onMounted(async () => {
  await getWallet().then(res => {
    myWallet.value = res;
    console.log('myWallet.value', myWallet.value);
  });
  await getAuthor().then(res => {
    if (res.creatorLeave === 1) {
      authorLevel.value = 5;
    } else if (res.creatorLeave === 2) {
      authorLevel.value = 30;
    } else if (res.creatorLeave === 3) {
      authorLevel.value = 330;
    } else if (res.creatorLeave === 4) {
      authorLevel.value = 1830;
    } else {
      authorLevel.value = 4830;
    }
  });
});

const hotTipsVisible = ref(false);
const hotTips = () => {
  hotTipsVisible.value = !hotTipsVisible.value;
};

const taxVisible = ref(false);
const taxDesc = () => {
  taxVisible.value = true;
};

const keyboardVisible = ref(false);
// 提现 -> 吊起提现金额
const withdrawal = () => {
  taxVisible.value = false;
  keyboardVisible.value = true;
};
</script>

<template>
  <PageLayout background="var(--secondaryBackground)">
    <template #navigationBarCenter>
      <div>{{ $t('mine.my_wallet') }}</div>
    </template>

    <div class="w-full px-16 pt-16 pb-37">
      <!-- 提现金额 -->
      <div
        class="mt-16 w-full aspect-358/159 rounded-16 bg-gradient-to-b from-#0075FF to-#53A1FF px-16 py-12"
      >
        <div class="flex items-center justify-between">
          <div class="flex items-end gap-4">
            <div class="text-14 lh-20 font-500 text-#FFF">
              {{ $t('my_wallet.available_withdrawal_amount') }}
            </div>
            <div class="text-10 lh-16 text-#FFF/70">
              {{ $t('my_wallet.content_earnings') }}
            </div>
          </div>
          <div class="flex items-center gap-6 relative" @click="hotTips">
            <div class="text-12 lh-17 text-#FFF/70">
              {{ $t('my_wallet.hot_value') }}
            </div>
            <div class="i-mingcute:question-fill w-13 h-13 text-#FFF/70"></div>
            <div
              :class="[
                'absolute top-27 right-0 px-11 pb-7 pt-11 bg-#FFF rounded-8 min-w-170 transition-opacity ease-in-out duration-300  shadow-[0px_0px_10px_0px_#00000026]',
                hotTipsVisible
                  ? 'opacity-100'
                  : 'opacity-0 pointer-events-none',
              ]"
            >
              <div class="relative">
                <div
                  class="absolute top-[-20px] left-2/3 w-0 h-0 b-b-10 b-b-solid b-b-#FFF b-x-10 b-x-solid b-x-transparent"
                ></div>
                <div class="text-12 lh-17 text-$tertiaryText">
                  {{ '🔥 ' + $t('my_wallet.hot_tips') }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="flex items-center justify-between mt-16">
          <div class="flex items-end gap-4 text-#FFF">
            <div class="text-24 lh-45 font-600">
              {{ $t('money_symbol') }}
            </div>
            <div class="text-40 lh-56 font-600">
              {{ myWallet?.moneyContent ?? 0 }}
            </div>
          </div>

          <div
            class="min-w-86 min-h-32 py-6 px-16 rounded-16 bg-#FFF flex-center text-14 lh-20 text-$positive font-500"
            @click="taxDesc"
          >
            {{ $t('my_wallet.go_withdraw') }}
          </div>
        </div>

        <div class="w-full h-1 mt-12 bg-#FFF/10"></div>

        <div class="flex items-center justify-between mt-12">
          <div class="flex items-center">
            <div class="text-13 lh-18 text-#FFF/70">
              {{ $t('my_wallet.total_withdraw') }}
            </div>
            <div class="text-13 lh-18 text-#FFF font-600 ml-4">
              {{ $t('money_symbol') + (myWallet?.totalWithdraw ?? 0) }}
            </div>
          </div>

          <div class="flex items-center">
            <div class="text-13 lh-18">🧧</div>
            <div class="text-13 lh-18 text-#FFF/70">
              {{ $t('my_wallet.red_packet_reward') }}
            </div>
            <div class="text-13 lh-18 text-#FFF font-600 ml-4">
              {{ $t('money_symbol') + (authorLevel ?? 0) }}
            </div>
          </div>
        </div>
      </div>

      <!-- 金额明细 -->
      <div class="mt-20 pt-12 px-16 pb-18 w-full rounded-16 bg-#FFF">
        <div class="text-16 lh-22 font-600">
          {{ $t('my_wallet.amount_details') }}
        </div>

        <div class="mt-12 w-full h-1 bg-$primaryDivider"></div>

        <div class="mt-12 flex flex-col gap-16">
          <div
            class="h-46 flex items-start gap-10"
            v-for="(item, index) in myWallet?.amountDetails"
            :key="item.submitTime"
          >
            <div
              class="w-36 h-36 rounded-50% flex-center"
              :style="{
                background: item.walletDetailColor,
              }"
            >
              <img :src="item.walletDetailIcon" alt="" class="w-21 h-21" />
            </div>
            <div
              class="w-[calc(100%-46px)] flex items-center justify-between pb-8"
              :class="{
                'b-b-1 b-b-solid b-b-$primaryDivider':
                  index !== myWallet?.amountDetails.length! - 1,
              }"
            >
              <div class="flex flex-col justify-start">
                <div class="text-14 lh-20 font-500">
                  {{ item.walletDetailTitle }}
                </div>
                <div class="text-12 lh-17 text-$quaternaryText">
                  {{ item.submitTime }}
                </div>
              </div>
              <div
                class="text-14 lh-13 font-500"
                :class="[
                  item.walletDetailMoney > 0 ? ' text-#08D36A' : 'text-$red',
                ]"
              >
                {{ item.walletDetailMoney }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="fixed bottom-14 right-13">
        <hintCard></hintCard>
      </div>
    </div>
  </PageLayout>

  <individualTaxInstructionPopup
    :visible="taxVisible"
    @update:visible="withdrawal"
  ></individualTaxInstructionPopup>

  <keyboard
    :visible="keyboardVisible"
    @update:visible="keyboardVisible = $event"
    :moneyContent="myWallet?.moneyContent ?? 0"
  ></keyboard>
</template>

<style scoped lang="scss"></style>
