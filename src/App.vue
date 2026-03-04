<script setup>
import { RouterView } from 'vue-router'

// IMPORTAR IMÁGENES
import koheartsNeutral from '@/assets/characters/kohearts/kohearts_neutral.png'
import whiterabbitNeutral from '@/assets/characters/whiterabbit/whiterabbit_neutral.png'
import cheshirecatNeutral from '@/assets/characters/cheshirecat/cheshirecat_neutral.png'
import koheartsTalk from '@/assets/characters/kohearts/kohearts_talk.png'
import whiterabbitTalk from '@/assets/characters/whiterabbit/whiterabbit_talk.png'
import cheshirecatTalk from '@/assets/characters/cheshirecat/cheshirecat_talk.png'

import dungeon1 from '@/assets/backgrounds/dungeon1.png'
import dungeon2 from '@/assets/backgrounds/dungeon2.png'

// PERSONAJES
const characters = [
  { 
    id: "kohearts", 
    name: "King of Hearts", 
    portraits: { 
      neutral: koheartsNeutral,
      talk: koheartsTalk
    }
  },
  { 
    id: "whiterabbit", 
    name: "White Rabbit", 
    portraits: { 
      neutral: whiterabbitNeutral,
      talk: whiterabbitTalk
    }
  },
  { 
    id: "cheshirecat", 
    name: "Cheshire Cat", 
    portraits: { 
      neutral: cheshirecatNeutral,
      talk: cheshirecatTalk
    }
  }
]

// DIÁLOGOS
const dialogues = [
  {
    id: "d1",
    characterIds: ["kohearts", "whiterabbit"],
    speakerId: "kohearts",
    background: dungeon1,
    pages: [
      "Bienvenido al reino subterráneo.", 
      "Soy King of Hearts, protector de estas tierras."
    ]
  },

  {
    id: "d2",
    characterIds: ["kohearts", "whiterabbit"],
    speakerId: "whiterabbit",
    background: dungeon1,
    pages: [
      "Bienvenido al reino subterráneo.",
      "Soy White Rabbit, tu guía en este lugar."
    ]
  },

  {
    id: "d3",
    characterIds: ["cheshirecat"],
    speakerId: "cheshirecat",
    background: dungeon2,
    pages: [
      "Soy el misterioso Cheshire Cat."
    ]
  },

  {
    id: "d4",
    characterIds: ["cheshirecat"],
    speakerId: null,
    background: dungeon2,
    pages: [
      "..."
    ]
  }
]
</script>

<template>
  <RouterView 
      :characters="characters"
      :dialogues="dialogues"
    />
</template>