<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// RECIBIMOS LOS DATOS DESDE APP
const props = defineProps({
    characters: Array,
    dialogues: Array,
    speed: { type: Number, default: 50 } // velocidad del typewriter en ms
})

// ESTADO
const stepIndex = ref(0)
const textPageIndex = ref(0)
const displayedText = ref('')
const typewriterEnabled = ref(true)
let typewriterTimeout = null

// PRECARGAR IMÁGENES AL INICIAR
onMounted(() => {
    props.characters.forEach(character => {
    const imgNeutral = new Image()
    imgNeutral.src = character.portraits.neutral
    const imgTalk = new Image()
    imgTalk.src = character.portraits.talk
    })

    props.dialogues.forEach(dialogue => {
    const img = new Image()
    img.src = dialogue.background
    })

    startCurrentText()
})

// DIÁLOGO ACTUAL
const currentDialogue = computed(() => props.dialogues[stepIndex.value])
const currentText = computed(() => currentDialogue.value.pages[textPageIndex.value])

// TYPEWRITER
function typeWriter(text) {
    displayedText.value = ''
    let i = 0

    function addLetter() {
    if (i < text.length) {
        displayedText.value += text[i]
        i++
        typewriterTimeout = setTimeout(addLetter, props.speed)
    }
    }

    addLetter()
}

function startCurrentText() {
    clearTimeout(typewriterTimeout)
    typeWriter(currentText.value)
}

// AVANZAR
function next() {
    if (textPageIndex.value < currentDialogue.value.pages.length - 1) {
    textPageIndex.value++
    } else if (stepIndex.value < props.dialogues.length - 1) {
    stepIndex.value++
    textPageIndex.value = 0
    }
    startCurrentText()
}

// RETROCEDER
function prev() {
    if (textPageIndex.value > 0) {
        textPageIndex.value--
    } else if (stepIndex.value > 0) {
        stepIndex.value--
        textPageIndex.value = props.dialogues[stepIndex.value].pages.length - 1
    }
    startCurrentText()
}

// Reiniciar typewriter si cambia de página automáticamente
watch([stepIndex, textPageIndex], () => startCurrentText())
</script>

<template>
<div class="min-h-screen relative bg-black text-white overflow-hidden">

    <!-- Fondo dinámico -->
    <div
        class="absolute inset-0 bg-cover bg-center"
        :style="{ backgroundImage: `url(${currentDialogue.background})` }"
    ></div>

    <!-- Personajes en escena -->
    <div
        class="absolute bottom-32 w-full flex justify-center items-end gap-4 px-4 md:gap-12"
    >
        <img
            v-for="(charId, index) in currentDialogue.characterIds"
            :key="charId"
            :src="props.characters.find(c => c.id === charId).portraits[
                currentDialogue.speakerId === charId ? 'talk' : 'neutral'
            ]"
            class="transition-all duration-300
                w-1/2
                sm:w-2/5
                md:w-1/3
                lg:w-1/4"
        />
    </div>

    <!-- Caja de diálogo -->
    <div class="absolute bottom-0 w-full bg-gray-900 bg-opacity-95 p-6 md:p-8 border-t-4 border-yellow-500 shadow-lg font-serif h-48 flex flex-col justify-between">

        <!-- Nombre del hablante -->
        <div class="font-bold text-lg md:text-2xl text-yellow-400 tracking-wide min-h-8">
            {{ currentDialogue.speakerId ? props.characters.find(c => c.id === currentDialogue.speakerId).name : '' }}
        </div>

        <!-- Texto -->
        <div class="mt-2 md:mt-3 text-base md:text-lg text-white">
            {{ typewriterEnabled ? displayedText : currentText }}
        </div>

        <!-- Botón animación -->
        <button
            @click="typewriterEnabled = !typewriterEnabled"
            :class="[
                'absolute top-2 right-2 px-2 py-1 text-xs shadow transition-colors duration-200',
                typewriterEnabled 
                    ? 'bg-yellow-500 text-black' 
                    : 'bg-gray-700 text-white hover:bg-yellow-500 hover:text-black'
            ]"
            :title="typewriterEnabled ? 'Desactivar animación' : 'Activar animación'"
        >
            Anim
        </button>

        <!-- Botones -->
        <div class="flex justify-end mt-4 space-x-2">
            <button 
                @click="prev" 
                class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200"
            >
                Atrás
            </button>
            <button 
                @click="next" 
                class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200"
            >
                Siguiente
            </button>
        </div>
    </div>

</div>
</template>