<template>
  <div class="p-6">
    <!-- Header -->
    <div class="text-center mb-6">
      <h2 class="text-2xl font-bold text-gray-800">Questionnaire</h2>
      <div v-if="currentStep > 0 && currentStep <= questions.length" class="w-full bg-gray-200 rounded-full h-2 mt-4">
        <div class="bg-indigo-600 h-2 rounded-full transition-all duration-300"
          :style="{ width: `${(currentStep / questions.length) * 100}%` }"></div>
      </div>
    </div>

    <!-- Welcome Screen -->
    <div v-if="currentStep === 0" class="text-center">
      <div class="mb-6">
        <div class="w-16 h-16 bg-indigo-100 rounded-full flex items-center justify-center mx-auto mb-4">
          <svg class="w-8 h-8 text-indigo-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z">
            </path>
          </svg>
        </div>
        <h3 class="text-lg font-semibold text-gray-800 mb-2">Bizning so'rovnomamizga xush kelibsiz!</h3>
        <p class="text-gray-600">Iltimos, bir nechta savollarga javob berishga vaqt ajrating. Bu faqat bir daqiqa vaqt oladi.</p>
      </div>
      <button @click="startQuestionnaire"
        class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-8 rounded-lg transition-colors duration-200">
        Start
      </button>
    </div>

    <!-- Question Screen -->
    <div v-else-if="currentStep <= questions.length" class="space-y-6">
      <!-- Question -->
      <div>
        <h3 class="text-xl font-semibold text-gray-800 mb-4">
          {{ questions[currentStep - 1].question }}
        </h3>

        <!-- Options -->
        <div class="space-y-3">
          <label v-for="(option, index) in questions[currentStep - 1].options" :key="index"
            class="flex items-center p-4 border-2 rounded-lg cursor-pointer transition-all duration-200 hover:bg-gray-50"
            :class="{
              'border-indigo-500 bg-indigo-50': answers[currentStep - 1] === option,
              'border-gray-200': answers[currentStep - 1] !== option
            }">
            <input type="radio" :name="`question-${currentStep}`" :value="option" v-model="answers[currentStep - 1]"
              class="sr-only">
            <div class="w-5 h-5 border-2 rounded-full mr-3 flex items-center justify-center">
              <div v-if="answers[currentStep - 1] === option" class="w-3 h-3 bg-indigo-600 rounded-full"></div>
            </div>
            <span class="text-gray-700">{{ option }}</span>
          </label>
        </div>
      </div>

      <!-- Navigation Buttons -->
      <div class="flex justify-between pt-4">
        <button @click="previousStep" :disabled="currentStep === 1"
          class="px-6 py-2 text-gray-600 border border-gray-300 rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-50">
          Orqaga
        </button>
        <button @click="nextStep" :disabled="!answers[currentStep - 1]"
          class="px-6 py-2 bg-indigo-600 text-white rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed hover:bg-indigo-700">
          {{ currentStep === questions.length ? 'Tugatish' : 'Keyingisi' }}
        </button>
      </div>
    </div>

    <!-- Confirmation Screen -->
    <div v-else-if="currentStep === questions.length + 1" class="text-center">
      <div class="mb-6">
        <div class="w-16 h-16 bg-yellow-100 rounded-full flex items-center justify-center mx-auto mb-4">
          <svg class="w-8 h-8 text-yellow-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M8.228 9c.549-1.165 2.03-2 3.772-2 2.21 0 4 1.343 4 3 0 1.4-1.278 2.575-3.006 2.907-.542.104-.994.54-.994 1.093m0 3h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z">
            </path>
          </svg>
        </div>
        <h3 class="text-lg font-semibold text-gray-800 mb-2">Ishonchingiz komilmi?</h3>
        <p class="text-gray-600">Iltimos, javoblaringizni yubormoqchi ekanligingizni tasdiqlang.</p>
      </div>
      <div class="flex justify-center space-x-4">
        <button @click="goBackToLastQuestion"
          class="px-6 py-2 text-gray-600 border border-gray-300 rounded-lg transition-colors duration-200 hover:bg-gray-50">
          Ortga qaytish
        </button>
        <button @click="confirmSubmit"
          class="px-6 py-2 bg-indigo-600 text-white rounded-lg transition-colors duration-200 hover:bg-indigo-700">
          Ha, yuborish
        </button>
      </div>
    </div>

    <!-- Completion Screen -->
    <div v-else class="text-center">
      <div class="mb-6">
        <div class="w-16 h-16 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-4">
          <svg class="w-8 h-8 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
          </svg>
        </div>
        <h3 class="text-lg font-semibold text-gray-800 mb-2">Rahmat!</h3>
        <p class="text-gray-600">So'rovnomada qatnashganingiz uchun rahmat!</p>
      </div>
      <button @click="closeModal"
        class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-8 rounded-lg transition-colors duration-200">
        Yopish
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'QuestionnaireModal',
  props: {
    isOpen: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      currentStep: 0,
      answers: [],
      questions: [
        {
          question: 'Sizning sevimli rangingiz qanday?',
          options: ['Qizil', 'Ko`k', 'Yashil', 'Boshqa']
        },
        {
          question: 'Sizning yoshingiz nechida?',
          options: ['18 yoshdan past', '18–24', '25–34', '35+']
        },
        {
          question: 'Biz haqimizda qayerdan bildingiz?',
          options: ['Instagramdan', 'Do`stimdan', 'Googledan', 'Boshqa']
        },
        {
          question: 'Bizni tavsiya qilasizmi?',
          options: ['Albatta', 'O`ylab ko`raman', 'Yo`q']
        }
      ]
    }
  },
  methods: {
    startQuestionnaire() {
      this.currentStep = 1
      this.answers = new Array(this.questions.length).fill('')
    },
    nextStep() {
      if (this.currentStep === this.questions.length) {
        this.currentStep = this.questions.length + 1
      } else {
        this.currentStep++
      }
    },
    confirmSubmit() {
      this.$emit('submit', this.answers)
      this.currentStep = this.questions.length + 2
    },
    goBackToLastQuestion() {
      this.currentStep = this.questions.length
    },
    previousStep() {
      if (this.currentStep > 1) {
        this.currentStep--
      }
    },
    closeModal() {
      this.currentStep = 0
      this.answers = []
      this.$emit('close')
    }
  },
  watch: {
    isOpen(newVal) {
      if (!newVal) {
        this.currentStep = 0
        this.answers = []
      }
    }
  }
}
</script>