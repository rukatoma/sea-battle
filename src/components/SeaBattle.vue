<script setup>
import { ref, watch } from 'vue';

const ships = ref({ x1: 4, x2: 3, x3: 2, x4: 1});
const mustPlaceShip = ref(10);
const selfGrid = ref([[[]*9]*11]);
const gridOfPlacedShips = ref([]);
const gameReady = ref(false);
const logs = ref([]);

const hitBlast = (e) => {
    e.target.classList.remove('missed');
    e.target.classList.add('blasted');
    let pos = Number(e.target.getAttribute('pos')) + 1;
    let col = pos % 11;
    let row = ((pos - col) / 11) + 1;
    if(col === 0) { col = 11; row = row - 1 }
    logs.value.push('Hit blast in position: row-' + row + ', col-' + col);
}

const blankBlast = (e) => {
    if(e.target.classList.contains('blasted') === false && e.target.classList.contains('missed') === false) {
        e.target.classList.add('missed');
    }
}

let endGame = 0;

const checkBlast = (e) => {
    let pos = e.target.getAttribute('pos');
    if(gameReady.value) {
        for(let i = 0; i < 10; i++) {
            for(let j = 0; j < gridOfPlacedShips.value[i].length; j++) {
                if(gridOfPlacedShips.value[i][j] === Number(pos)) {
                    console.log('Hit!');
                    hitBlast(e);
                    gridOfPlacedShips.value[i].splice(j, 1);
                    if(gridOfPlacedShips.value[i].length === 0) {
                        gridOfPlacedShips.value.splice(i, 1);
                        logs.value.push('A ship was destroyed!');
                    }
                    if(gridOfPlacedShips.value.length === 0) {
                        logs.value.push('End game!');
                    }
                } else if(gridOfPlacedShips.value[i][j] !== Number(pos)) {
                    console.log('Miss!');
                    blankBlast(e);
                }
            }
        }
    } else {
        console.log('Игра еще не началась');
    }
}

const setNotPlace = (arr, pos, num) => {
    let first, second = false;
    if(arr[pos - 1] !== undefined) {
        if((pos) % 11 !== 0) {
            arr[pos - 1].classList.add('not-place');
            first = true;
        }
    }
    if(arr[pos + num] !== undefined) {
        if((pos + num) % 11 !== 0) {
            arr[pos + num].classList.add('not-place');
            second = true;
        }
    }
    for(let i = -1; i < num + 1; i++) {
        if((i === -1 && first) || (i === num && second)) {
            if(arr[pos + i - 11] !== undefined) {
                arr[pos + i - 11].classList.add('not-place');
            }
            if(arr[pos + i + 11] !== undefined) {
                arr[pos + i + 11].classList.add('not-place');
            }
        } else if(i > -1 && i < num) {
            if(arr[pos + i - 11] !== undefined) {
                arr[pos + i - 11].classList.add('not-place');
            }
            if(arr[pos + i + 11] !== undefined) {
                arr[pos + i + 11].classList.add('not-place');
            }
        }
    }
}

const ready = () => {
    gameReady.value = true;
    logs.value.push('Game starts!');
    let fields = document.getElementsByClassName('self-field');
    for(let i = 0; i < fields.length; i++) 
    {
        if(fields[i].classList.contains('not-place')) {
            fields[i].classList.remove('not-place');
        }
    }
}

const setShip = (e) => {
    let pos = 0;
    let fields = document.getElementsByClassName('self-field');
    switch(mustPlaceShip.value) {
        case 10: 
            pos = e.target.getAttribute('pos');
            for(let i = 0; i < fields.length; i++) {
                if(fields[i].getAttribute('pos') === pos) {
                    if(Math.ceil((i + 1) / 11) === Math.ceil((i + 2) / 11) && 
                       Math.ceil((i + 1) / 11) === Math.ceil((i + 3) / 11) &&
                       Math.ceil((i + 1) / 11) === Math.ceil((i + 4) / 11)) 
                    {
                        setNotPlace(fields, i, 4);
                        fields[i].classList.add('ship-peace', 'first-peace');
                        fields[i + 1].classList.add('ship-peace', 'middle-peace');
                        fields[i + 2].classList.add('ship-peace', 'middle-peace');
                        fields[i + 3].classList.add('ship-peace', 'last-peace');
                        gridOfPlacedShips.value.push([i, i+1, i+2, i+3]);
                    } else {
                        return;
                    }
                }
            }
            if(mustPlaceShip.value === 10) {
                mustPlaceShip.value = 9;
                ships.value.x4 = 0;
                break;
            }
        case 9: case 8: 
            pos = e.target.getAttribute('pos');
            for(let i = 0; i < fields.length; i++) {
                if(fields[i].getAttribute('pos') === pos) {
                    if(fields[i + 2].classList.contains('not-place') === false &&
                       fields[i + 1].classList.contains('not-place') === false && 
                       fields[i].classList.contains('not-place') === false &&
                       fields[i].classList.contains('ship-peace') === false) 
                    {
                        if(Math.ceil((i + 1) / 11) === Math.ceil((i + 2) / 11) && 
                           Math.ceil((i + 1) / 11) === Math.ceil((i + 3) / 11)) 
                        {
                            setNotPlace(fields, i, 3);
                            fields[i].classList.add('ship-peace', 'first-peace');
                            fields[i + 1].classList.add('ship-peace', 'middle-peace');
                            fields[i + 2].classList.add('ship-peace', 'last-peace');
                            gridOfPlacedShips.value.push([i, i+1, i+2]);
                        } else {
                            return;
                        }
                    } else {
                        console.log('Placcing is not allowed');
                        return;
                    }
                }
            }
            if(mustPlaceShip.value === 9) {
                mustPlaceShip.value = 8;
                ships.value.x3 = 1;
                break;
            } else if(mustPlaceShip.value === 8) {
                mustPlaceShip.value = 7;
                ships.value.x3 = 0;
                break;
            }
        case 7: case 6: case 5: 
            pos = e.target.getAttribute('pos');
            for(let i = 0; i < fields.length; i++) {
                if(fields[i].getAttribute('pos') === pos) {
                    if(fields[i + 1].classList.contains('not-place') === false && 
                       fields[i].classList.contains('not-place') === false &&
                       fields[i].classList.contains('ship-peace') === false) 
                    {
                        if(Math.ceil((i + 1) / 11) === Math.ceil((i + 2) / 11)) 
                        {
                            setNotPlace(fields, i, 2);
                            fields[i].classList.add('ship-peace', 'first-peace');
                            fields[i + 1].classList.add('ship-peace', 'last-peace');
                            gridOfPlacedShips.value.push([i, i+1]);
                        } else {
                            return;
                        }
                    } else {
                        console.log('Placcing is not allowed');
                        return;
                    }
                }
            }
            if(mustPlaceShip.value === 7) {
                mustPlaceShip.value = 6;
                ships.value.x2 = 2;
                break;
            } else if(mustPlaceShip.value === 6) {
                mustPlaceShip.value = 5;
                ships.value.x2 = 1;
                break;
            } else if(mustPlaceShip.value === 5) {
                mustPlaceShip.value = 4;
                ships.value.x2 = 0;
                break;
            }
        case 4: case 3: case 2: case 1: 
            pos = e.target.getAttribute('pos');
            for(let i = 0; i < fields.length; i++) {
                if(fields[i].getAttribute('pos') === pos) {
                    if(fields[i].classList.contains('not-place') === false &&
                       fields[i].classList.contains('ship-peace') === false) 
                    {
                        setNotPlace(fields, i, 1);
                        fields[i].classList.add('ship-peace');
                        gridOfPlacedShips.value.push([i]);
                    } else {
                        console.log('Placcing is not allowed');
                        return;
                    }
                }
            }
            if(mustPlaceShip.value === 4) {
                mustPlaceShip.value = 3;
                ships.value.x1 = 3;
                break;
            } else if(mustPlaceShip.value === 3) {
                mustPlaceShip.value = 2;
                ships.value.x1 = 2;
                break;
            } else if(mustPlaceShip.value === 2) {
                mustPlaceShip.value = 1;
                ships.value.x1 = 1;
                break;
            } else if(mustPlaceShip.value === 1) {
                mustPlaceShip.value = 0;
                ships.value.x1 = 0;
                ready();
                break;
            }
        case 0:
            console.log('Все корабли на поле');
            console.log('Игра началась!');
            break;
    }
}

logs.value.push('Please, place your ships!');
</script>

<template>
    <h1 class="text-center uppercase font-bold text-5xl text-slate-900 mb-20">Self destroy sea battle</h1>
    <div class="w-full h-24 p-5 bg-blue-200 mb-10 rounded-xl overflow-y-scroll" @onChange="scrollTop = 9999">
        <p v-for="item in logs.slice().reverse()">{{ item }}</p>
    </div>
    <article class="flex gap-10">
        <div>
            <h2 class="uppercase text-center mb-3 text-bold text-2xl text-slate-600">Battle grid</h2>
            <div class="battle-grid outself grid grid-cols-11 grid-rows-9 bg-blue-400 p-1 rounded-xl">
                <div
                    v-for="(item, index) in 99" 
                    :key="index" 
                    :pos="index"
                    type="outself-field"
                    @click="checkBlast($event)"
                    class="field outself-field relative w-14 h-14 m-1 bg-blue-100 rounded-md cursor-pointer duration-100 hover:shadow-inner" 
                />
            </div>
        </div>
        <div>
            <h2 class="uppercase text-center mb-3 text-bold text-2xl text-slate-600">Ships grid</h2>
            <div class="battle-grid self grid grid-cols-11 grid-rows-9 p-1 bg-blue-400 rounded-xl mb-6">
                <div 
                    v-for="(item, index) in 99" 
                    :key="index" 
                    :pos="index"
                    type="self-field"
                    @click="setShip($event)"
                    class="field self-field self relative w-14 h-14 m-1 bg-blue-300 rounded-md cursor-pointer duration-100 hover:shadow-inner" 
                />
            </div>
            <h3 class="mb-5">Ship places: left to right</h3>
            <div class="flex gap-10">
                <div class="flex flex-col items-center">
                    <div class="ship cursor-pointer flex items-center">
                        <div v-for="item in 4" class="ship-peace w-8 h-8 bg-slate-900 border-slate-500 border-2" />
                    </div>
                    <span class="text-lg">x{{ ships.x4 }}</span>
                </div>
                <div class="flex flex-col items-center">
                    <div class="ship cursor-pointer flex items-center">
                        <div v-for="item in 3" class="ship-peace w-8 h-8 bg-slate-900 border-slate-500 border-2" />
                    </div>
                    <span class="text-lg">x{{ ships.x3 }}</span>
                </div>
                <div class="flex flex-col items-center">
                    <div class="ship cursor-pointer flex items-center">
                        <div v-for="item in 2" class="ship-peace w-8 h-8 bg-slate-900 border-slate-500 border-2" />
                    </div>
                    <span class="text-lg">x{{ ships.x2 }}</span>
                </div>
                <div class="flex flex-col items-center">
                    <div class="ship cursor-pointer flex items-center">
                        <div v-for="item in 1" class="ship-peace w-8 h-8 bg-slate-900 border-slate-500 border-2" />
                    </div>
                    <span class="text-lg">x{{ ships.x1 }}</span>
                </div>
            </div>
        </div>
    </article>
</template>

<style scoped>
.battle-grid.outself .field:hover {
    transform: scale(.9);
}

.battle-grid.outself .field.blasted:hover, .battle-grid.outself .field.missed:hover {
    transform: none;
}

.field.blasted, .field.missed {
    background: transparent;
}

.field.blasted:hover, .field.missed:hover {
    box-shadow: none;
}

.field.self:hover {
    box-shadow: none;
}

.field.blasted::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 60px;
    height: 60px;
    background: url('/blast.png') 0 0 no-repeat;
    background-position: center;
    background-size: contain;
    animation: blast 500ms ease;
}

.field.missed::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 44px;
    height: 44px;
    background: url('/splash.png') 0 0 no-repeat;
    background-position: center;
    background-size: contain;
    animation: miss 500ms ease;
}

@keyframes blast {
    0% {
        width: 20px;
        height: 20px;
    }

    100% {
        width: 60px;
        height: 60px;
    }
}

@keyframes miss {
    0% {
        width: 20px;
        height: 20px;
    }

    75% {
        width: 54px;
        height: 54px;
    }

    100% {
        width: 44px;
        height: 44px;
    }
}

.ship-peace {
    position: relative;
    border-top: none;
    border-bottom: none;
    border-left: none;
    border-right-width: 2px;
}

.field.ship-peace {
    border-right: none;
    background-color: transparent;
}

.ship-peace::after {
    content: '';
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: rgb(15 23 42);
    top: -3px;
    left: 50%;
    border-radius: 2px;
    transform: translate(-50%, 0);
}

.ship-peace:first-child::after, .ship-peace:last-child::after {
    top: 0;
}

.ship-peace:last-child {
    border-top-right-radius: 6px;
    border-end-end-radius: 16px;
    border-right: none;
    border-left: none;
}

.ship-peace:first-child {
    border-top-left-radius: 6px;
    border-end-start-radius: 16px;
    border-left: none;
}

@keyframes ship {
    25% {
        top: 47%;
    }
    75% {
        top: 51%;
    }
    100% {
        top: 50%;
    }
}

@keyframes ship-p {
    25% {
        top: calc(-20px * .51);
    }
    75% {
        top: calc(-20px * .47);
    }
    100% {
        top: calc(-20px * .50);
    }
}

.field.ship-peace::after {
    animation: ship 2000ms infinite;
}

.field.ship-peace::before {
    animation: ship-p 2000ms infinite;
}

.field.ship-peace:after {
    content: '';
    z-index: 999;
    position: absolute;
    width: 100%;
    height: 36px;
    background-color: rgb(15 23 42);
    top: 50%;
    left: 50%;
    border-radius: 6px 6px 20px 20px;
    transform: translate(-50%, -50%);
}

.field.ship-peace.first-peace::after {
    transform: translate(-42%, -50%);
    border-radius: 6px 0 0 20px;
    width: 56px;
}

.field.ship-peace.last-peace::after {
    transform: translate(-58%, -50%);
    border-radius: 0 6px 20px 0;
    width: 56px;
}

.field.ship-peace.middle-peace::after {
    width: 64px;
    border-radius: 0;
}

.field.ship-peace.middle-peace::before {
    content: '';
    width: 30px;
    height: 10px;
    position: absolute;
    top: -10px;
    left: 50%;
    transform: translate(-50%, 100%);
    background-color: rgb(15 23 42);
    border-radius: 6px 6px 0 0;
}
.field.not-place {
    background-color: pink;
}
</style>
