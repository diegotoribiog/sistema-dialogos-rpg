<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { useStepper } from "@vueuse/core";

const props = defineProps({
  characters: Array,
  dialogues: Array,
  speed: { type: Number, default: 50 },
});

// STEPPER DE DIÁLOGOS
const dialogueStepper = useStepper(props.dialogues.map((d) => d.id));

// STEPPER DE PÁGINAS (reactivo al diálogo actual)
const currentDialogue = computed(() =>
  props.dialogues.find((d) => d.id === dialogueStepper.current.value),
);

//STEPPER DE PÁGINA ACTUAL DENTRO DEL DIÁLOGO
const pageStepper = useStepper(
  computed(() => currentDialogue.value.pages.map((_, i) => i)),
);

// TYPEWRITER
const displayedText = ref("");
const typewriterEnabled = ref(true);
let typewriterTimeout = null;

const currentText = computed(
  () => currentDialogue.value.pages[pageStepper.current.value],
);

function typeWriter(text) {
  clearTimeout(typewriterTimeout);
  displayedText.value = "";
  let i = 0;
  function addLetter() {
    if (i < text.length) {
      displayedText.value += text[i++];
      typewriterTimeout = setTimeout(addLetter, props.speed);
    }
  }
  addLetter();
}

function startCurrentText() {
  typeWriter(currentText.value);
}

// AVANZAR
function next() {
  if (!pageStepper.isLast.value) {
    pageStepper.goToNext();
  } else if (!dialogueStepper.isLast.value) {
    dialogueStepper.goToNext();
    // Reiniciar páginas al primer paso del nuevo diálogo
    pageStepper.goTo(0);
  }
}

// RETROCEDER
function prev() {
  if (!pageStepper.isFirst.value) {
    pageStepper.goToPrevious();
  } else if (!dialogueStepper.isFirst.value) {
    dialogueStepper.goToPrevious();
    // Ir a la última página del diálogo anterior
    pageStepper.goTo(currentDialogue.value.pages.length - 1);
  }
}

// PRECARGAR IMÁGENES
onMounted(() => {
  props.characters.forEach((c) => {
    new Image().src = c.portraits.neutral;
    new Image().src = c.portraits.talk;
  });
  props.dialogues.forEach((d) => {
    new Image().src = d.background;
  });
  startCurrentText();
});

watch(
  [() => dialogueStepper.current.value, () => pageStepper.current.value],
  () => {
    startCurrentText();
  },
);
</script>

<template>
  <div class="min-h-screen relative bg-black text-white overflow-hidden">
    <!-- Fondo dinámico -->
    <div
      class="absolute inset-0 bg-cover bg-center transition-all duration-500"
      :style="{ backgroundImage: `url(${currentDialogue.background})` }"
    ></div>

    <!-- Personajes en escena -->
    <div
      class="absolute bottom-32 w-full flex justify-center items-end gap-4 px-4 md:gap-12"
    >
      <img
        v-for="charId in currentDialogue.characterIds"
        :key="charId"
        :src="
          props.characters.find((c) => c.id === charId).portraits[
            currentDialogue.speakerId === charId ? 'talk' : 'neutral'
          ]
        "
        class="transition-all duration-300 w-1/2 sm:w-2/5 md:w-1/3 lg:w-1/4"
      />
    </div>

    <!-- Caja de diálogo -->
    <div
      class="absolute bottom-0 w-full bg-gray-900 bg-opacity-95 p-6 md:p-8 border-t-4 border-yellow-500 shadow-lg font-serif h-48 flex flex-col justify-between"
    >
      <!-- Nombre del hablante -->
      <div
        class="font-bold text-lg md:text-2xl text-yellow-400 tracking-wide min-h-8"
      >
        {{
          currentDialogue.speakerId
            ? props.characters.find((c) => c.id === currentDialogue.speakerId)
                .name
            : ""
        }}
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
            : 'bg-gray-700 text-white hover:bg-yellow-500 hover:text-black',
        ]"
      >
        Anim
      </button>

      <!-- Botones de navegación -->
      <div class="flex justify-between items-center mt-4">
        <!-- Indicador de progreso -->
        <span class="text-xs text-gray-400">
          {{ dialogueStepper.index.value + 1 }}/{{ props.dialogues.length }} ·
          pág {{ pageStepper.index.value + 1 }}/{{
            currentDialogue.pages.length
          }}
        </span>

        <div class="flex space-x-2">
          <button
            @click="prev"
            :disabled="
              dialogueStepper.isFirst.value && pageStepper.isFirst.value
            "
            class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200 disabled:opacity-30 disabled:cursor-not-allowed"
          >
            Atrás
          </button>
          <button
            @click="next"
            :disabled="dialogueStepper.isLast.value && pageStepper.isLast.value"
            class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200 disabled:opacity-30 disabled:cursor-not-allowed"
          >
            Siguiente
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
