<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';

let cookies = ref(0);
let totalCookiesBaked = ref(0);
let showAchievement = ref(false);
let currentAchievement = ref({});
let achievementQueue = ref([]);

let buildings = ref([
    { name: 'Cursor', price: 15, count: 0, cps: 5, icon: '👆' },
    { name: 'Grandma', price: 100, count: 0, cps: 100, icon: '👵' },
    { name: 'Farm', price: 1100, count: 0, cps: 1000, icon: '🌾' },
    { name: 'Factory', price: 12000, count: 0, cps: 5000, icon: '🏭' },
    { name: 'Mine', price: 130000, count: 0, cps: 10000, icon: '⛏️' },
    { name: 'Bank', price: 1400000, count: 0, cps: 20000, icon: '🏦' },
    { name: 'Temple', price: 20000000, count: 0, cps: 780000, icon: '🏛️' },
    { name: 'Wizard Tower', price: 330000000, count: 0, cps: 44000, icon: '🧙' },
]);

let achievements = ref([
    { id: 'firstClick', name: 'First Click', description: 'Click the cookie for the first time', earned: false, condition: () => totalClicks.value >= 1 },
    { id: 'tenClicks', name: 'Cookie Beginner', description: 'Click the cookie 10 times', earned: false, condition: () => totalClicks.value >= 10 },
    { id: 'hundredClicks', name: 'Cookie Enthusiast', description: 'Click the cookie 100 times', earned: false, condition: () => totalClicks.value >= 100 },
    { id: 'thousandClicks', name: 'Cookie Fanatic', description: 'Click the cookie 1,000 times', earned: false, condition: () => totalClicks.value >= 1000 },
    { id: 'firstCookie', name: 'First Cookie', description: 'Bake your first cookie', earned: false, condition: () => totalCookiesBaked.value >= 1 },
    { id: 'hundredCookies', name: 'Hundred Cookies', description: 'Bake 100 cookies', earned: false, condition: () => totalCookiesBaked.value >= 100 },
    { id: 'thousandCookies', name: 'Baker', description: 'Bake 1,000 cookies', earned: false, condition: () => totalCookiesBaked.value >= 1000 },
    { id: 'millionCookies', name: 'Cookie Millionaire', description: 'Bake 1,000,000 cookies', earned: false, condition: () => totalCookiesBaked.value >= 1000000 },
    { id: 'firstGrandma', name: 'Grandma\'s Helper', description: 'Hire your first grandma', earned: false, condition: () => buildings.value[1].count >= 1 },
    { id: 'tenGrandmas', name: 'Grandma\'s Army', description: 'Hire 10 grandmas', earned: false, condition: () => buildings.value[1].count >= 10 },
    { id: 'firstFarm', name: 'Farmer', description: 'Build your first farm', earned: false, condition: () => buildings.value[2].count >= 1 },
]);


let totalClicks = ref(0);
let earnedAchievements = computed(() => achievements.value.filter(a => a.earned).length);


let cps = computed(() => {
    return buildings.value.reduce((total, building) => total + building.cps * building.count, 0);
});


const cpsInterval = setInterval(() => {
    const cookiesGenerated = cps.value / 10;
    cookies.value += cookiesGenerated;
    totalCookiesBaked.value += cookiesGenerated;
    document.title = `🍪 ${Math.floor(cookies.value)} cookies`;
    checkAchievements();
}, 100);

function formatNumber(num) {
    if (num >= 1000000000) {
        return (num / 1000000000).toFixed(1) + ' billion';
    } else if (num >= 1000000) {
        return (num / 1000000).toFixed(1) + ' million';
    } else if (num >= 1000) {
        return (num / 1000).toFixed(1) + ' thousand';
    } else {
        return num.toFixed(1);
    }
}

function clickCookie() {
    cookies.value++;
    totalCookiesBaked.value++;
    totalClicks.value++;
}

function buyBuilding(building) {
    cookies.value -= building.price;
    building.price = Math.ceil(building.price * 1.15);
    building.count++;
}

function checkAchievements() {
    for (let achievement of achievements.value) {
        if (!achievement.earned && achievement.condition()) {
            achievement.earned = true;
            achievementQueue.value.push(achievement);
            if (!showAchievement.value) {
                showNextAchievement();
            }
        }
    }
}

function showNextAchievement() {
    if (achievementQueue.value.length > 0) {
        currentAchievement.value = achievementQueue.value.shift();
        showAchievement.value = true;
        setTimeout(() => {
            showAchievement.value = false;
        
            setTimeout(() => {
                if (achievementQueue.value.length > 0) {
                    showNextAchievement();
                }
            }, 500);
        }, 3000);
    }
}

onUnmounted(() => {
    clearInterval(cpsInterval);
});
</script>

<template>
    <div class="game-container">
        <div class="achievement-notification" v-if="showAchievement" :class="{ 'show': showAchievement }">
            <div class="achievement-inner">
                <div class="achievement-icon">🏆</div>
                <div class="achievement-content">
                    <h3>Achievement Unlocked!</h3>
                    <h4>{{ currentAchievement.name }}</h4>
                    <p>{{ currentAchievement.description }}</p>
                </div>
            </div>
        </div>

        <div class="columns">
            <div class="column is-3 cookie-panel">
                <h1 class="is-size-2 cookie-counter">
                    <b>🍪 {{ formatNumber(cookies) }}</b>
                </h1>
                <h3 class="is-size-4 cps-counter">
                    <b>{{ formatNumber(cps) }} per second</b>
                </h3>
                <div class="cookie-container">
                    <img @click="clickCookie" class="cookie-image" src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" alt="Cookie" />
                </div>
                <div class="stats-panel">
                    <p><b>Recently unlocked:</b> {{ earnedAchievements }}/{{ achievements.length }}</p>
                </div>
            </div>

            <div class="column is-9 buildings-panel">
                <h2 class="is-size-4 has-text-centered">Buildings</h2>
                <div class="buildings-container">
                    <button 
                        class="building-button" 
                        v-for="building in buildings" 
                        :key="building.name"
                        :disabled="cookies < building.price" 
                        @click="buyBuilding(building)"
                    >
                        <div class="building-info">
                            <span class="building-icon">{{ building.icon }}</span>
                            <span class="building-name">{{ building.name }}</span>
                            <span class="building-count">#{{ building.count }}</span>
                            <span class="building-price">🍪{{ formatNumber(building.price) }}</span>
                        </div>
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
.game-container {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    height: 100vh;
    background-color: #f5f0e6;
    overflow: hidden;
}

.cookie-panel {
    background-color: #8b5e34;
    color: #fff;
    padding: 20px;
    text-align: center;
    height: 100vh;
    display: flex;
    flex-direction: column;
}

.cookie-counter {
    margin-bottom: 5px;
    color: #ffdf7e;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.cps-counter {
    color: #ffc373;
    margin-bottom: 20px;
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.cookie-container {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
}

.cookie-image {
    width: 80%;
    max-width: 250px;
    cursor: pointer;
    transition: transform 0.1s;
}

.cookie-image:hover {
    transform: scale(1.05);
}

.cookie-image:active {
    transform: scale(0.95);
}

.stats-panel {
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    padding: 15px;
    margin-top: 20px;
    color: #fff9e0;
}

.main-content {
    background-color: #c17e3a;
    color: #fff;
    padding: 20px;
    height: 100vh;
    overflow-y: auto;
}

.stats-overview {
    margin-top: 20px;
}

.stats-cards {
    display: flex;
    justify-content: space-around;
    flex-wrap: wrap;
    margin-top: 30px;
}

.stat-card {
    background-color: rgba(255, 255, 255, 0.15);
    border-radius: 10px;
    padding: 20px;
    text-align: center;
    width: 30%;
    min-width: 150px;
    margin-bottom: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.stat-card h3 {
    color: #ffdf7e;
    font-size: 18px;
    margin-bottom: 10px;
}

.stat-card p {
    color: #fff;
    font-size: 24px;
    font-weight: bold;
}

.achievement-icon {
    font-size: 24px;
    margin-right: 10px;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 50%;
}

.achievement-details h4 {
    font-size: 16px;
    margin-bottom: 5px;
    color: #ffdf7e;
}

.achievement-details p {
    font-size: 12px;
    color: #fff9e0;
}

.buildings-panel {
    background-color: #644024;
    color: #fff;
    padding: 15px;
    height: 100vh;
    overflow-y: auto;
}

.buildings-container {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    padding: 10px;
}

.building-button {
    width: calc(25% - 10px);
    background-color: #8b5e34;
    border: none;
    border-radius: 6px;
    color: #fff;
    padding: 15px 10px;
    cursor: pointer;
    transition: background-color 0.2s;
    text-align: left;
}

@media (max-width: 1200px) {
    .building-button {
        width: calc(33.333% - 10px);
    }
}

@media (max-width: 900px) {
    .building-button {
        width: calc(50% - 10px);
    }
}

@media (max-width: 600px) {
    .building-button {
        width: 100%;
    }
}

.building-button:hover:not([disabled]) {
    background-color: #a06d3c;
}

.building-button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}

.building-info {
    display: flex;
    flex-direction: column;
}

.building-icon {
    font-size: 18px;
    margin-right: 5px;
}

.building-name {
    font-weight: bold;
    margin-bottom: 3px;
}

.building-count {
    font-size: 12px;
    color: #ffdf7e;
}

.building-price {
    font-size: 12px;
    color: #ffc373;
}

.achievement-notification {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%) translateY(-100px);
    z-index: 1000;
    background-color: rgba(255, 223, 126, 0.9);
    color: #644024;
    border-radius: 10px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
    padding: 15px;
    transition: transform 0.5s ease-in-out, opacity 0.3s ease-in-out;
    width: 300px;
    opacity: 0;
}

.achievement-notification.show {
    transform: translateX(-50%) translateY(0);
    opacity: 1;
}

.achievement-inner {
    display: flex;
    align-items: center;
}

.achievement-notification .achievement-icon {
    font-size: 30px;
    background: none;
    color: #644024;
}

.achievement-notification h3 {
    font-size: 18px;
    margin-bottom: 5px;
    color: #644024;
}

.achievement-notification h4 {
    font-size: 16px;
    margin-bottom: 5px;
    color: #8b5e34;
}

.achievement-notification p {
    font-size: 14px;
    color: #644024;
}

.columns {
    margin: 0;
    height: 100%;
}

.column {
    padding: 0;
}
</style>