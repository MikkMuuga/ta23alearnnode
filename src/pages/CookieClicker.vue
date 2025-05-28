<script setup>
import { computed, onUnmounted, ref } from 'vue';

let cookies = ref(0);

let buildings = ref([
    { name: 'Cursor', price: 15, count: 0, cps: 0.1, multiplier: 1 },
    { name: 'Grandma', price: 100, count: 0, cps: 1, multiplier: 1 },
    { name: 'Farm', price: 1000, count: 0, cps: 10, multiplier: 1 },
    { name: 'Factory', price: 10000, count: 0, cps: 100, multiplier: 1 },
]);

//keskmisesse ritta 2 korda võimsamad tooted
let upgrades = ref([
    { name: 'Double cursor', building: 'Cursor', price: 150, multiplier: 2, purchased: false },
    { name: 'Grandmas2', building: 'Grandma', price: 1000, multiplier: 2, purchased: false },
    { name: 'Crazy farm', building: 'Farm', price: 10000, multiplier: 2, purchased: false },
    { name: 'Company', building: 'Factory', price: 100000, multiplier: 2, purchased: false },
]);

let cps = computed(() => {
    return buildings.value.reduce((total, building) => total + building.cps * building.count * building.multiplier, 0);
});

const cpsInterval = setInterval(() => {
    cookies.value += cps.value;
    document.title = `🍪 ${cookies.value} cookies`;
}, 1000);

function buyBuilding(building) {
    cookies.value -= building.price;
    building.price += Math.ceil(building.price * 0.15);
    building.count++;
}

function buyUpgrade(upgrade) {
    cookies.value -= upgrade.price;
    upgrade.purchased = true;
    
    const building = buildings.value.find(b => b.name === upgrade.building);
    if (building) {
        building.multiplier *= upgrade.multiplier;
    }
}
onUnmounted(() => {
    clearInterval(cpsInterval);
});
</script>
<template>
    <div class="columns">
        <div class="column is-3 has-background-primary has-text-centered">
            <h1 class="is-size-1"><b>Cookies: {{ cookies.toFixed(1) }}</b></h1>
            <h3 class="is-size-3"><b>CPS: {{ +cps.toFixed(1) }}</b></h3>
            <figure class="image is-square m-5">
                <img @click="cookies++" class="is-rounded" src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" />
            </figure>
        </div>
        <div class="column is-7 has-background-info">
            <h2 class="is-size-2 has-text-centered has-text-white mb-4"><b>Upgrades</b></h2>
            <div class="columns is-multiline">
                <div class="column is-6" v-for="upgrade in upgrades" :key="upgrade.name">
                    <button 
                        class="button is-medium is-fullwidth is-success" 
                        :disabled="cookies < upgrade.price || upgrade.purchased" 
                        @click="buyUpgrade(upgrade)"
                        v-if="!upgrade.purchased"
                    >
                        <div>
                            <div><strong>{{ upgrade.name }}</strong></div>
                            <div>{{ upgrade.building }} x{{ upgrade.multiplier }}</div>
                            <div>🍪{{ upgrade.price }}</div>
                        </div>
                    </button>
                    <div class="notification is-success" v-else>
                        <strong>{{ upgrade.name }}</strong> ✓
                    </div>
                </div>
            </div>
        </div>
        <div class="column is-2 has-background-warning">
            <button class="button is-large is-fullwidth is-primary mb-2" v-for="building in buildings" :disabled="cookies<building.price" @click="buyBuilding(building)" >
                <div>
                    <div>{{ building.name }}</div>
                    <div>🍪{{ building.price }}</div>
                    <div>#{{ building.count }}</div>
                    <div v-if="building.multiplier > 1" class="is-size-7">x{{ building.multiplier }}</div>
                </div>
            </button>
        </div>
    </div>
</template>