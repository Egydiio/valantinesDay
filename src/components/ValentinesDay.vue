<template>
  <div class="min-h-screen bg-gradient-to-br from-pink-100 via-blue-50 to-pink-200 overflow-hidden relative">
    <!-- Corações flutuantes de fundo -->
    <div class="fixed inset-0 pointer-events-none">
      <div
          v-for="heart in floatingHearts"
          :key="heart.id"
          class="absolute text-pink-300 opacity-30 animate-float"
          :style="{
          left: heart.x + '%',
          top: heart.y + '%',
          animationDelay: heart.delay + 's',
          fontSize: heart.size + 'px'
        }"
      >
        💖
      </div>
    </div>

    <!-- Corações que seguem o mouse -->
    <div class="fixed inset-0 pointer-events-none z-20">
      <div
          v-for="heart in mouseHearts"
          :key="heart.id"
          class="absolute text-pink-400 text-sm animate-mouseHeart"
          :style="{
          left: heart.x + 'px',
          top: heart.y + 'px',
          transform: 'translate(-50%, -50%)'
        }"
      >
        💖
      </div>
    </div>

    <!-- Header -->
    <header class="relative z-10 p-6 text-center">
      <h1 class="text-4xl md:text-6xl font-bold bg-gradient-to-r from-pink-500 to-blue-500 bg-clip-text text-transparent animate-pulse">
        💕 Dia dos Namorados 💕
      </h1>
    </header>

    <!-- Conteúdo Principal -->
    <main class="relative z-10 container mx-auto px-4 py-8">
      <!-- Seção do Texto Romântico -->
      <div class="max-w-4xl mx-auto text-center mb-12">
        <div class="bg-white/80 backdrop-blur-sm rounded-3xl p-8 shadow-2xl border border-pink-200">
          <h2 class="text-3xl md:text-4xl font-bold text-pink-600 mb-6 animate-bounce">
            Uma Mensagem Especial Para Você ❤️
          </h2>

          <div class="text-lg md:text-xl text-gray-700 leading-relaxed mb-8">
            <p v-if="showText" class="typewriter">
              {{ displayedText }}
              <span class="cursor">|</span>
            </p>
          </div>

          <button
              v-if="!textComplete && !showText"
              @click="startTyping"
              class="bg-gradient-to-r from-pink-500 to-blue-500 text-white px-8 py-4 rounded-full text-xl font-semibold hover:scale-110 transform transition-all duration-300 shadow-lg hover:shadow-xl"
          >
            💌 Clique para ler a mensagem
          </button>

          <div v-if="textComplete" class="mt-8">
            <button
                @click="showCarouselAndScroll"
                class="bg-gradient-to-r from-blue-500 to-pink-500 text-white px-10 py-5 rounded-full text-xl font-semibold hover:scale-110 transform transition-all duration-300 shadow-lg hover:shadow-xl animate-pulse"
            >
              ✨ Ver Nossas Memórias ✨
            </button>
          </div>
        </div>
      </div>

      <!-- Carrossel de Fotos e Vídeos -->
      <div
          ref="carouselRef"
          v-if="showCarousel"
          class="carousel-container max-w-7xl mx-auto"
          :class="{ 'animate-slideUp': showCarousel }"
      >
        <div class="bg-white/90 backdrop-blur-sm rounded-3xl p-8 shadow-2xl border border-blue-200">
          <h3 class="text-3xl font-bold text-center text-blue-600 mb-8">
            💫 Nossos Momentos Especiais 💫
          </h3>

          <div class="relative">
            <!-- Carrossel -->
            <div class="overflow-hidden rounded-2xl shadow-xl">
              <div
                  class="flex transition-transform duration-500 ease-in-out"
                  :style="{ transform: `translateX(-${currentSlide * 100}%)` }"
              >
                <div
                    v-for="(item, index) in carouselItems"
                    :key="index"
                    class="w-full flex-shrink-0 relative"
                >
                  <!-- Renderiza imagem ou vídeo dependendo do tipo -->
                  <img
                      v-if="item.type === 'image'"
                      :src="item.src"
                      :alt="item.alt"
                      class="w-full h-[500px] md:h-[600px] lg:h-[700px] object-contain bg-gray-100"
                      @error="handleImageError"
                  />

                  <div v-else-if="item.type === 'video'" class="w-full h-[500px] md:h-[600px] lg:h-[700px] bg-black flex items-center justify-center">
                    <video
                        :ref="el => { if (el) videoRefs[index] = el }"
                        class="w-full h-full object-contain"
                        controls
                        :poster="item.poster"
                        @play="handleVideoPlay(index)"
                    >
                      <source :src="item.src" type="video/mp4">
                      Seu navegador não suporta vídeos.
                    </video>
                  </div>

                  <div class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/70 to-transparent p-6">
                    <p class="text-white text-xl font-semibold">{{ item.caption }}</p>
                  </div>
                </div>
              </div>
            </div>

            <!-- Controles do Carrossel -->
            <button
                @click="prevSlide"
                class="absolute left-4 top-1/2 transform -translate-y-1/2 bg-pink-500 text-white p-3 rounded-full hover:bg-pink-600 transition-colors shadow-lg"
            >
              ❮
            </button>
            <button
                @click="nextSlide"
                class="absolute right-4 top-1/2 transform -translate-y-1/2 bg-pink-500 text-white p-3 rounded-full hover:bg-pink-600 transition-colors shadow-lg"
            >
              ❯
            </button>

            <!-- Indicadores -->
            <div class="flex justify-center mt-6 space-x-2">
              <button
                  v-for="(item, index) in carouselItems"
                  :key="index"
                  @click="goToSlide(index)"
                  class="w-3 h-3 rounded-full transition-colors"
                  :class="currentSlide === index ? 'bg-pink-500' : 'bg-gray-300'"
              ></button>
            </div>
          </div>
        </div>
      </div>

      <!-- Seção de Surpresa Final -->
      <div v-if="showCarousel" class="mt-16 text-center animate-fadeIn">
        <div class="bg-gradient-to-r from-pink-500 to-blue-500 rounded-3xl p-8 text-white shadow-2xl">
          <h3 class="text-3xl font-bold mb-4">💝 Surpresa Final 💝</h3>
          <p class="text-xl mb-6">
            Cada momento ao seu lado é um presente. Você é minha pessoa favorita no mundo inteiro!
          </p>
          <button
              @click="showHeartExplosion"
              :disabled="isHeartAnimationActive"
              class="bg-white text-pink-500 px-8 py-4 rounded-full text-xl font-bold hover:scale-110 transform transition-all duration-300 shadow-lg disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"
          >
            💖 {{ isHeartAnimationActive ? '...' : 'Clique aqui para uma surpresa!' }} 💖
          </button>
        </div>
      </div>
    </main>

    <!-- Explosão de Corações -->
    <div v-if="heartExplosion" class="fixed inset-0 pointer-events-none z-50">
      <div
          v-for="heart in explosionHearts"
          :key="heart.id"
          class="absolute text-6xl animate-heartExplosion"
          :style="{
          left: heart.x + '%',
          top: heart.y + '%',
          animationDelay: heart.delay + 's'
        }"
      >
        {{ heart.emoji }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

// Estados reativos
const showText = ref(false)
const displayedText = ref('')
const textComplete = ref(false)
const showCarousel = ref(false)
const currentSlide = ref(0)
const heartExplosion = ref(false)
const isHeartAnimationActive = ref(false)
const videoRefs = ref({})
const carouselRef = ref(null)

// Corações que seguem o mouse
const mouseHearts = ref([])
let heartId = 0

// Texto romântico
const fullText = "Meu amor, cada dia ao seu lado é uma nova aventura cheia de alegria e carinho. Você ilumina minha vida de uma forma única e especial. Neste Dia dos Namorados, quero que saiba o quanto você significa para mim. Você é minha melhor amiga, minha companheira de vida e o amor da minha vida. Obrigado por todos os momentos incríveis que vivemos juntos! 💕"

// Itens do carrossel (fotos e vídeos)
import foto1 from '/public/images/1.jpg'
import foto2 from '/public/images/2.jpg'
import foto3 from '/public/images/3.jpg'
import foto4 from '/public/images/4.jpg'
import foto5 from '/public/images/5.jpg'
import foto6 from '/public/images/6.jpg'
import foto7 from '/public/images/7.jpg'
import foto8 from '/public/images/8.jpg'
import foto9 from '/public/images/9.jpg'
import foto10 from '/public/images/10.jpg'
import foto11 from '/public/images/11.jpg'
import foto12 from '/public/images/12.jpg'
import foto13 from '/public/images/13.jpg'
import foto14 from '/public/images/14.jpg'
import foto15 from '/public/images/15.jpg'
import foto16 from '/public/images/16.jpg'
import foto17 from '/public/images/17.jpg'
import foto18 from '/public/images/18.jpg'

// Itens do carrossel (fotos e vídeos)
const carouselItems = ref([
  {
    type: 'image',
    src: foto5,
    caption: 'Primeira foto postada 💕'
  },
  {
    type: 'image',
    src: foto6,
    caption: 'No parque juntos 🌳'
  },
  {
    type: 'image',
    src: foto7,
    caption: 'A foto que mais amo de nós dois ❤️'
  },
  {
    type: 'image',
    src: foto8,
    caption: 'Pensa em uma pessoa que ama tirar foto! 📸'
  },
  {
    type: 'image',
    src: foto10,
    caption: 'Ida ao shopping 🛍️'
  },
  {
    type: 'image',
    src: foto12,
    caption: 'Milésimos churrasquinho 🍖'
  },
  {
    type: 'image',
    src: foto13,
    caption: 'Na igreja juntos ⛪'
  },
  {
    type: 'image',
    src: foto9,
    caption: 'Na casa da Nadia 🏡'
  },
  {
    type: 'image',
    src: foto11,
    caption: 'Pensa em um casal bonito! 😍'
  },
  {
    type: 'image',
    src: foto14,
    caption: 'Foto do seu aniversário 🎉'
  },
  {
    type: 'image',
    src: foto15,
    caption: 'Aline nos seus joguinhos 🎮'
  },
  {
    type: 'image',
    src: foto16,
    caption: 'Nossa formatura da escola 🎓'
  },
  {
    type: 'image',
    src: foto4,
    caption: 'Precisamos de mais viagens juntos! 🚗'
  },
  {
    type: 'image',
    src: foto17,
    caption: 'Quando vamos nos proximo show? 🎤'
  },
  {
    type: 'image',
    src: foto3,
    caption: 'Praia é sempre bom! 🏖️'
  },
  {
    type: 'image',
    src: foto18,
    caption: 'Olha esse casal lindo! 😘'
  },
  {
    type: 'image',
    src: foto2,
    caption: 'Sofrendo pelo Galo 🐔'
  },
  {
    type: 'image',
    src: foto1,
    caption: 'Nossa viagem mais importante! ✈️'
  }
])

// Corações flutuantes
const floatingHearts = ref([])
const explosionHearts = ref([])

// Funções
const handleImageError = (event) => {
  console.log('Erro ao carregar imagem:', event.target.src)
  // Substitui por uma imagem placeholder se der erro
  event.target.src = '/placeholder.svg?height=400&width=800&text=Imagem+não+encontrada'
}

const startTyping = () => {
  if (showText.value) return

  showText.value = true
  let index = 0
  const typeInterval = setInterval(() => {
    displayedText.value = fullText.slice(0, index)
    index++
    if (index > fullText.length) {
      clearInterval(typeInterval)
      textComplete.value = true
    }
  }, 50)
}

const nextSlide = () => {
  pauseCurrentVideo()
  currentSlide.value = (currentSlide.value + 1) % carouselItems.value.length
}

const prevSlide = () => {
  pauseCurrentVideo()
  currentSlide.value = currentSlide.value === 0 ? carouselItems.value.length - 1 : currentSlide.value - 1
}

const goToSlide = (index) => {
  pauseCurrentVideo()
  currentSlide.value = index
}

const pauseCurrentVideo = () => {
  Object.values(videoRefs.value).forEach(video => {
    if (video && !video.paused) {
      video.pause()
    }
  })
}

const handleVideoPlay = (playingIndex) => {
  Object.entries(videoRefs.value).forEach(([index, video]) => {
    if (Number(index) !== playingIndex && video && !video.paused) {
      video.pause()
    }
  })
}

const showHeartExplosion = () => {
  if (isHeartAnimationActive.value) return

  isHeartAnimationActive.value = true
  heartExplosion.value = true
  explosionHearts.value = []

  const heartEmojis = ['💖', '💕', '💗', '💓', '💝', '💘', '❤️', '🧡', '💛', '💚', '💙', '💜']

  for (let i = 0; i < 50; i++) {
    explosionHearts.value.push({
      id: i,
      x: Math.random() * 100,
      y: Math.random() * 100,
      delay: Math.random() * 2,
      emoji: heartEmojis[Math.floor(Math.random() * heartEmojis.length)]
    })
  }

  setTimeout(() => {
    heartExplosion.value = false
    isHeartAnimationActive.value = false
  }, 5000)
}

const createFloatingHearts = () => {
  for (let i = 0; i < 20; i++) {
    floatingHearts.value.push({
      id: i,
      x: Math.random() * 100,
      y: Math.random() * 100,
      delay: Math.random() * 5,
      size: 20 + Math.random() * 20
    })
  }
}

const createMouseHeart = (x, y) => {
  const heart = {
    id: heartId++,
    x: x,
    y: y,
    opacity: 1
  }

  mouseHearts.value.push(heart)

  setTimeout(() => {
    const index = mouseHearts.value.findIndex(h => h.id === heart.id)
    if (index > -1) {
      mouseHearts.value.splice(index, 1)
    }
  }, 1000)
}

const handleMouseMove = (event) => {
  if (Math.random() > 0.9) {
    createMouseHeart(event.clientX, event.clientY)
  }
}

let carouselInterval
const startCarouselAutoPlay = () => {
  carouselInterval = setInterval(() => {
    if (showCarousel.value) {
      nextSlide()
    }
  }, 4000)
}

watch(() => currentSlide.value, (newSlide) => {
  const currentItem = carouselItems.value[newSlide]
  if (currentItem.type === 'video') {
    clearInterval(carouselInterval)
  } else {
    clearInterval(carouselInterval)
    startCarouselAutoPlay()
  }
})

onMounted(() => {
  createFloatingHearts()
  startCarouselAutoPlay()
  document.addEventListener('mousemove', handleMouseMove)
})

onUnmounted(() => {
  clearInterval(carouselInterval)
  document.removeEventListener('mousemove', handleMouseMove)
})

const showCarouselAndScroll = () => {
  showCarousel.value = true

  // Aguarda um pouco para o elemento aparecer no DOM e depois faz o scroll
  setTimeout(() => {
    if (carouselRef.value) {
      carouselRef.value.scrollIntoView({
        behavior: 'smooth',
        block: 'start'
      })
    }
  }, 100)
}
</script>

<style scoped>
@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(180deg); }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes heartExplosion {
  0% {
    transform: scale(0) rotate(0deg);
    opacity: 1;
  }
  50% {
    transform: scale(1.5) rotate(180deg);
    opacity: 0.8;
  }
  100% {
    transform: scale(0) rotate(360deg);
    opacity: 0;
  }
}

@keyframes mouseHeart {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(0);
  }
  50% {
    opacity: 0.8;
    transform: translate(-50%, -50%) scale(1);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.5) translateY(-20px);
  }
}

.animate-float {
  animation: float 6s ease-in-out infinite;
}

.animate-slideUp {
  animation: slideUp 0.8s ease-out;
}

.animate-fadeIn {
  animation: fadeIn 1s ease-in;
}

.animate-heartExplosion {
  animation: heartExplosion 3s ease-out forwards;
}

.animate-mouseHeart {
  animation: mouseHeart 1s ease-out forwards;
}

.typewriter {
  font-family: 'Courier New', monospace;
}

.cursor {
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.carousel-container {
  animation: slideUp 0.8s ease-out;
}

button:hover {
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}

.bg-gradient-to-br {
  background-size: 400% 400%;
  animation: gradientShift 8s ease infinite;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
</style>
