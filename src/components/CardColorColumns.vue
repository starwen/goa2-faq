<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { cardColumn } from '@/helper/cards';
import HeroCard from './HeroCard.vue';
import SearchInput from './SearchInput.vue';

const props = withDefaults(
    defineProps<{
        cards: Card[];
        searchable?: boolean;
    }>(),
    {
        cards: () => [],
        searchable: true,
    }
);

const filterName = ref('');
const filterTier = ref<string[]>([]);

const tiers = computed(() => {
    return props.cards
        .map(c => c.tier ?? '\xA0')
        .filter((v, i, a) => a.indexOf(v) === i)
        .sort((a, b) => {
            if (a === '\xA0' || b === 'H') return -1;
            if (b === '\xA0' || a === 'H') return 1;
            return a.localeCompare(b);
        });
});

// 监听 tiers（下拉框选项），在数据加载完的第一时间自动勾选除了 'H' 之外的所有选项
watch(tiers, (newTiers) => {
    // 增加一个标记，确保只有在页面第一次完全加载、且没有任何勾选时才赋默认值
    if (filterTier.value.length === 0 && newTiers.length > 0) {
        filterTier.value = newTiers.filter(t => t !== 'H');
    }
}, { immediate: true });

const columns = computed(() => props.cards.slice(0).reduce<Card[][]>(
    (acc, current) => {
        if (props.searchable) {
            if (filterName.value && !current.name.toLowerCase().includes(filterName.value.toLowerCase())) {
                return acc;
            }
            // if (filterTier.value.length && !filterTier.value.includes(current.tier ?? '\xA0')) {
            //     return acc;
            // }
            // 修改意义：只要当前卡牌的等级没有在勾选列表（filterTier）里，就直接过滤掉！
            if (!filterTier.value.includes(current.tier ?? '\xA0')) {
                return acc;
            }
        }

        acc[cardColumn(current)].push(current);
        return acc;
    },
    [[], [], [], []] as Card[][]
).filter(c => c && c.length));

</script>

<template>
    <div>
        <div class="search-input-container">
            <SearchInput v-if="searchable" v-model:name="filterName" v-model:choices="filterTier" :options="tiers"
                :placeholder="$t('app.button.search')" />
        </div>

        <div class="card-column-container">
            <div v-for="(column, idx) in columns" :key="idx">
                <div v-for="card of column" :key="card.id">
                    <HeroCard :card="card" />
                </div>
            </div>
        </div>
    </div>

</template>

<style lang="scss">
.search-input-container {
    text-align: center;
    margin: .25em auto;
    width: 100%;
}

.card-column-container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    column-gap: 1.5em;
    row-gap: 3em;
    margin: 0 .15em;

    .hero-card-wrapper {
        margin: 1.5em auto;
    }

    @media (max-width: 1280px) {
        grid-template-columns: repeat(2, 1fr);
        column-gap: 2.5em;
    }

    @media (max-width: 720px) {
        column-gap: 1.5em;
    }

    @media (max-width: 580px) {
        grid-template-columns: 1fr;

        .hero-card-wrapper {
            margin: 1em auto;
        }
    }
}
</style>
