<template>
    <div>
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-2x-l font-bold">Juego de Rol.</h2>
            <button @click="$emit('logout')" class="bg-red-500 text-white px-4 py-2
            rounded hover:bg-red-600">Cerrar Sesión.</button>
        </div>

        <!--Pantalla de Inicio o Reinicio-->
        <div v-if="!gameStarted || gameOver" class="text-center">
            <h3 class="text-lg font-semibold mb-4">{{ gameOver ? 'Juego Terminado' : 'Bienvenido al juego' }}</h3>
            <p v-if="gameOver" class="mb-4" :class="winner === 'Jugador' ?
            'text-success' : 'text-danger'">
            {{ winner ? '${winner} ha ganado' : 'Ha terminado la partida' }}
        </p>

        <button @click="startNewGame" class="bg-blue-500 text-white px-4 py-2
        rounded hover:bg-blue-600">Empezar Nueva Partida</button>
        </div>

        <!--Juego-->
        <div v-else>
            <div class="mb-6">
                <h3 class="text-lg font-semibold mb-2">Salud de los jugadores: </h3>
                <div class="mb-4">
                    <p class="font-semibold">Jugador: {{playerHealth}}/100</p>
                    <div class="w-full bg-gray-200 rounded-full h-4">
                        <div :style="{width: playerHealthPercentage + '%'}"
                        :class="playerHealthClass" class="h-4 rounded-full">
                        </div>
                    </div>
                </div>
                <div>
                    <p class="font-semibold">Dragón: {{dragonHealthPercentage}}/100</p>
                    <div class="w-full bg-gray-200 rounded-full h-4">
                        <div :style="{width: dragonHealthPercentage + '%'}"
                        :class="dragonHealthClass" class="h-4 rounded-full">
                        </div>
                    </div>
                </div>
            </div>

            <!--Acciones del jugador-->
            <div class="mb-6">
                <h3 class="text-lg font-semibold mb-2">Acciones</h3>
                <div class="grid grid-cols-2 gap-2">
                <button @click="attack" class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600">
                    Atacar</button>
                <button @click="heal" :disabled="!canHeal" :class="{ 'bg-gray-400 cursor-not-allowed': !canHeal, 'bg-blue-500 hover:bg-blue-600': canHeal }" class="text-white p-2 rounded">
                    Curar</button>
                <button @click="specialAttack" :disabled="!canSpecialAttack" :class="{ 'bg-gray-400 cursor-not-allowed': !canSpecialAttack, 'bg-blue-500 hover:bg-blue-600': canSpecialAttack }" class="text-white p-2 rounded">
                    Ataque Especial</button>
                <button @click="flee" class="bg-red-500 text-white p-2 rounded hover:bg-red-600">
                    Huir</button>
                </div>
            </div>

            <!--Registro-->
            <div>
                <h3 class="text-lg font-semibold mb-2">Registro de lucha</h3>
                <div class="max-h-40 overflow-y-auto border p-2 rounded">
                    <p v-for="(message, index) in mensajesBatalla" :key="index"
                    class="text-sm">{{ message }}</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, defineProps, defineEmits, onMounted, onUnmounted} from 'vue';

const props = defineProps(['currentUser']);
const emit = defineEmits(['logout']);

const gameStarted = ref(false);
const gameOver = ref(false);
const winner = ref('');
const playerHealth = ref(100);
const dragonHealth = ref(100);
const mensajesBatalla = ref([]);
const Heal = ref(0);
const SpecialAttack = ref(0);

// Propiedades computadas
const playerHealthPercentage = computed(() => Math.max(0, playerHealth.value));
const dragonHealthPercentage = computed(() => Math.max(0, dragonHealth.value));
const playerHealthClass = computed(() => {
    if(playerHealth.value <= 20) return 'bg-red-500';
    if(dragonHealth.value <= 50) return 'bg-yellow-500';
    return 'bg-green-500';
});

const dragonHealthClass = computed(() => {
    if(dragonHealth.value <= 20) return 'bg-red-500';
    if(dragonHealth.value <= 50) return 'bg-yellow-500';
    return 'bg-green-500';
});
const canHeal = computed(() => heal.value === 0);
const canSpecialAttack = computed(() => SpecialAttack.value === 0);

// Generamos número aleatorio en un rango
const random = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;

// Verificamos si acabó el juego
const GameOver = () => {
    if(playerHealth.value <= 0){
        gameOver.value = true;
        winner.value = 'Dragon';
        mensajesBatalla.value.unshift('Ha ganado el Dragón');
        return true;
    }
    if(dragonHealth.value <= 0){
        gameOver.value = true;
        winner.value = true;
        winner.value = 'Jugador';
        mensajesBatalla.value.unshift('Has ganado');
        return true;
    }
    return false;
}

// Iniciar una nueva partida
const startNewGame = () => {
    gameStarted.value = true;
    gameOver.value = false;
    winner.value = '';
    playerHealth.value = 100;
    dragonHealth.value = 100;
    mensajesBatalla.value = [];
    Heal.value = 0;
    SpecialAttack.value = 0;
};

// Botón atacar
const attack = () => {
    if(gameOver.value) return;
    const damage = random(5, 10);
    dragonHealth.value = Math.max(0, dragonHealth.value - damage);
    mensajesBatalla.value.unshift('Has atacado al Dragón y causa ${damage} de daño');
    GameOver();
};

// Botón de ataque especial
const specialAttack = () => {
    if(gameOver.value || !canSpecialAttack.value) return;
    const damage = random(10, 20);
    dragonHealth.value = Math.max(0, dragonHealth.value - damage);
    mensajesBatalla.value.unshift('Has usado tu ataque especial y causa ${damage} de daño');
    SpecialAttack.value = 3;
    GameOver();
};

// Botón curar
const heal = () => {
    if(gameOver.value || !canHeal.value) return;
    const healAmount = random(5, 10);
    playerHealth.value = Math.min(100, playerHealth.value + healAmount);
    mensajesBatalla.value.unshift('Te has curado ${{healAmount}} de vida.');
    Heal.value = 3;
    GameOver();
};

// Botón Huir
const flee = () => {
    gameOver.value = true;
    winner.value = 'Dragón';
    mensajesBatalla.value.unshift('Has huido, ha ganado el Dragón.')
};

// Ataque del dragón
const dragonAttack = () => {
    if(gameOver.value) return;
    const damage = random(8, 15);
    playerHealth.value = Math.max(0, playerHealth.value - damage);
    mensajesBatalla.value.unshift('El Dragón te atacó y causa ${damage} de daño');

    if(Heal.value > 0) Heal.value -=1;
    if(SpecialAttack.value > 0) SpecialAttack.value -=1;
    GameOver();
}

let dragonAttackTimes = null;
onMounted(() => {
    dragonAttackTimes = setInterval(() => {
        if(gameStarted.value && !gameOver.value){
            dragonAttack();
        }
    }, 2000); // Cada segundo el dragón ataca
});

onUnmounted(() => {
    if(dragonAttackTimes){
        clearInterval(dragonAttackTimes);
    }
});

</script>

