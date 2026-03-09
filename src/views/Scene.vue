<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from "vue";

const props = defineProps({
  characters: Array,
  backgrounds: Array,
  dialogues: Array,
  speed: { type: Number, default: 50 },
});

// HELPERS
function getCharacter(id) {
  return props.characters.find((c) => c.id === id) ?? null;
}
function getBackground(id) {
  return props.backgrounds.find((b) => b.id === id) ?? null;
}

// ÍNDICES DE NAVEGACIÓN
const dialogueIdx = ref(0);
const utteranceIdx = ref(0);
const pageIdx = ref(0);

// ESTADO ACTUAL
const currentDialogue = computed(() => props.dialogues[dialogueIdx.value]);
const currentUtterance = computed(
  () => currentDialogue.value.utterances[utteranceIdx.value],
);
const currentText = computed(() => currentUtterance.value.texts[pageIdx.value]);

// LÍMITES
const isFirst = computed(
  () =>
    dialogueIdx.value === 0 && utteranceIdx.value === 0 && pageIdx.value === 0,
);
const isLast = computed(() => {
  const lastDialogue = dialogueIdx.value === props.dialogues.length - 1;
  const lastUtterance =
    utteranceIdx.value === currentDialogue.value.utterances.length - 1;
  const lastPage = pageIdx.value === currentUtterance.value.texts.length - 1;
  return lastDialogue && lastUtterance && lastPage;
});

// FONDO ACTIVO
const currentBackground = computed(() => {
  const bgId =
    currentUtterance.value.backgroundId ??
    currentDialogue.value.defaultBackgroundId;
  return getBackground(bgId)?.image ?? "";
});

// HABLANTE ACTIVO
const currentSpeaker = computed(() =>
  getCharacter(currentUtterance.value.characterId),
);

// CLASE DE POSICIÓN DE LA IMAGEN
const imagePositionClass = computed(() => {
  const pos = currentUtterance.value.position ?? "left";
  if (pos === "right") return "justify-end";
  if (pos === "center") return "justify-center";
  return "justify-start";
});

// TYPEWRITER
const displayedText = ref("");
const typewriterEnabled = ref(true);
let typewriterTimeout = null;

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

// SÍNTESIS DE VOZ
const speechEnabled = ref(true);

function speak(text) {
  window.speechSynthesis.cancel();
  if (!speechEnabled.value) return;

  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = "es-ES";
  utterance.rate = 1;
  utterance.pitch = 1;

  // BUSCAR VOZ EN ESPAÑOL SI ESTÁ DISPONIBLE
  const voices = window.speechSynthesis.getVoices();
  const spanishVoice = voices.find((v) => v.lang.startsWith("es"));
  if (spanishVoice) utterance.voice = spanishVoice;

  window.speechSynthesis.speak(utterance);
}

// ARRANCAR TYPEWRITER Y VOZ
function startCurrentText() {
  typeWriter(currentText.value);
  speak(currentText.value);
}

// NAVEGACIÓN
function next() {
  if (pageIdx.value < currentUtterance.value.texts.length - 1) {
    pageIdx.value++;
  } else if (utteranceIdx.value < currentDialogue.value.utterances.length - 1) {
    utteranceIdx.value++;
    pageIdx.value = 0;
  } else if (dialogueIdx.value < props.dialogues.length - 1) {
    dialogueIdx.value++;
    utteranceIdx.value = 0;
    pageIdx.value = 0;
  }
}

function prev() {
  if (pageIdx.value > 0) {
    pageIdx.value--;
  } else if (utteranceIdx.value > 0) {
    utteranceIdx.value--;
    pageIdx.value = currentUtterance.value.texts.length - 1;
  } else if (dialogueIdx.value > 0) {
    dialogueIdx.value--;
    utteranceIdx.value = currentDialogue.value.utterances.length - 1;
    pageIdx.value = currentUtterance.value.texts.length - 1;
  }
}

// PRECARGA DE IMÁGENES
onMounted(() => {
  props.characters.forEach((c) => {
    new Image().src = c.image;
  });
  props.backgrounds.forEach((b) => {
    new Image().src = b.image;
  });
  startCurrentText();
});

// CANCELAR VOZ AL DESMONTAR EL COMPONENTE
onUnmounted(() => {
  window.speechSynthesis.cancel();
});

// DISPARAR TYPEWRITER Y VOZ AL CAMBIAR TEXTO
watch(currentText, () => startCurrentText());
</script>

<template>
  <div class="min-h-screen relative bg-black text-white overflow-hidden">
    <!-- FONDO DINÁMICO -->
    <div
      class="absolute inset-0 bg-cover bg-center transition-all duration-500"
      :style="{ backgroundImage: `url(${currentBackground})` }"
    />

    <!-- PERSONAJE EN ESCENA -->
    <div
      v-if="currentSpeaker"
      :class="[
        'absolute bottom-32 w-full flex items-end px-8 md:px-16',
        imagePositionClass,
      ]"
    >
      <img
        :key="currentSpeaker.id"
        :src="currentSpeaker.image"
        class="transition-all duration-300 w-1/2 sm:w-2/5 md:w-1/3 lg:w-1/4"
      />
    </div>

    <!-- CAJA DE DIÁLOGO -->
    <div
      class="absolute bottom-0 w-full bg-gray-900 bg-opacity-95 p-6 md:p-8 border-t-4 border-yellow-500 shadow-lg font-serif h-48 flex flex-col"
    >
      <!-- NOMBRE DEL HABLANTE -->
      <div
        class="font-bold text-lg md:text-2xl text-yellow-400 tracking-wide min-h-8"
      >
        {{ currentSpeaker?.name ?? "" }}
      </div>

      <!-- TEXTO -->
      <div class="mt-2 md:mt-3 text-base md:text-lg text-white">
        {{ typewriterEnabled ? displayedText : currentText }}
      </div>

      <!-- BOTÓN ANIMACIÓN -->
      <button
        @click="typewriterEnabled = !typewriterEnabled"
        :class="[
          'absolute top-2 right-16 px-2 py-1 text-xs shadow transition-colors duration-200',
          typewriterEnabled
            ? 'bg-yellow-500 text-black'
            : 'bg-gray-700 text-white hover:bg-yellow-500 hover:text-black',
        ]"
      >
        Anim
      </button>

      <!-- BOTÓN VOZ -->
      <button
        @click="
          speechEnabled = !speechEnabled;
          !speechEnabled && window.speechSynthesis.cancel();
        "
        :class="[
          'absolute top-2 right-2 px-2 py-1 text-xs shadow transition-colors duration-200',
          speechEnabled
            ? 'bg-yellow-500 text-black'
            : 'bg-gray-700 text-white hover:bg-yellow-500 hover:text-black',
        ]"
      >
        Voz
      </button>

      <!-- NAVEGACIÓN -->
      <div class="absolute bottom-4 right-4 flex space-x-2">
        <button
          @click="prev"
          :disabled="isFirst"
          class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200 disabled:opacity-30 disabled:cursor-not-allowed"
        >
          Atrás
        </button>
        <button
          @click="next"
          :disabled="isLast"
          class="bg-gray-700 px-3 md:px-4 py-2 shadow hover:bg-yellow-500 hover:text-black transition-colors duration-200 disabled:opacity-30 disabled:cursor-not-allowed"
        >
          Siguiente
        </button>
      </div>
    </div>
  </div>
</template>
