<template>
  <div class="flex flex-col items-center justify-center min-h-[80vh]">
    <!-- Logo at the top, always above the card -->
    <div class="z-20 flex justify-center" style="margin-bottom: -2.5rem;">
      <img src="/logo.png" alt="Logo" class="w-20 h-20 rounded-full border-4 border-white shadow-lg bg-white object-contain" />
    </div>
    <!-- Card -->
    <div class="z-10 bg-white rounded-[2rem] shadow-xl w-[340px] max-w-full px-6 py-8 flex flex-col items-center">
      <!-- Welcome Screen -->
      <div v-if="currentStep === 0" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">Welcome to our survey!</h3>
        <p class="text-gray-600 text-center mb-8">Please take a moment to answer a few questions. It will only take a minute.</p>
        <button @click="startQuestionnaire"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          Start
        </button>
      </div>
      <!-- Question Screen -->
      <div v-else-if="currentStep <= questions.length" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">
          {{ questions[currentStep - 1].question }}
        </h3>
        <div class="flex flex-col gap-3 w-full mb-4">
          <button v-for="(option, index) in questions[currentStep - 1].options" :key="index"
            @click="selectOption(option)"
            class="bg-gray-100 hover:bg-[#FF2D6A] hover:text-white text-gray-700 font-medium py-3 rounded-full w-full transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]"
            :class="{ 'bg-[#FF2D6A] text-white': answers[currentStep - 1] === option }">
            {{ option }}
          </button>
        </div>
        <div class="w-full flex flex-col items-center">
          <input v-if="questions[currentStep - 1].allowCustom" v-model="customAnswer" placeholder="Enter your answer"
            class="w-full rounded-xl border border-gray-200 px-4 py-2 mt-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
        </div>
        <button @click="previousStep" :disabled="currentStep === 1"
          class="mt-4 w-full max-w-xs bg-[#FF2D6A] text-white font-semibold py-3 rounded-full transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          Back
        </button>
      </div>
      <!-- Confirmation Screen -->
      <div v-else-if="currentStep === questions.length + 1" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">Are you sure?</h3>
        <p class="text-gray-600 text-center mb-8">Please confirm that you want to submit your answers.</p>
        <div class="flex gap-4 w-full">
          <button @click="goBackToLastQuestion"
            class="w-1/2 bg-gray-100 text-gray-700 font-semibold py-3 rounded-full transition-colors duration-200 hover:bg-gray-200">
            Go Back
          </button>
          <button @click="confirmSubmit"
            class="w-1/2 bg-[#FF2D6A] text-white font-semibold py-3 rounded-full transition-colors duration-200 hover:bg-[#e0265c]">
            Yes, Submit
          </button>
        </div>
      </div>
      <!-- Completion Screen -->
      <div v-else class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">Thank you!</h3>
        <p class="text-gray-600 text-center mb-8">Thank you for participating in the survey.</p>
        <button @click="closeModal"
          class="w-full max-w-xs bg-[#FF2D6A] text-white font-semibold py-3 rounded-full transition-colors duration-200">
          Close
        </button>
      </div>
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
      customAnswer: '',
      questions: [
        {
          question: 'What should we focus on most when announcing the promotion?',
          options: ['Product', 'Number of products', 'Price and quality', 'Promotion terms', 'Brand'],
          allowCustom: true
        },
        {
          question: 'If the promotion message is broadcast on the radio, do you want it to be in the form of a jingle? This increases its audibility and memorability.',
          options: ['Yes', 'No difference'],
          allowCustom: true
        },
        {
          question: 'Is this product, production, brand, sales activity, etc. related to you?',
          options: ['Yes, I am the company manager', 'No, but I have authority', 'I am not related'],
          allowCustom: true
        },
        {
          question: 'Is this product, production, brand, sales activity, etc. related to you?',
          options: [],
          allowCustom: true
        }
      ]
    }
  },
  methods: {
    startQuestionnaire() {
      this.currentStep = 1
      this.answers = new Array(this.questions.length).fill('')
      this.customAnswer = ''
    },
    selectOption(option) {
      this.answers[this.currentStep - 1] = option
      this.customAnswer = ''
      setTimeout(() => {
        this.nextStep()
      }, 150) // Small delay for button feedback
    },
    nextStep() {
      if (this.questions[this.currentStep - 1].allowCustom && this.customAnswer) {
        this.answers[this.currentStep - 1] = this.customAnswer
        this.customAnswer = ''
      }
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
        this.customAnswer = ''
      }
    },
    closeModal() {
      this.currentStep = 0
      this.answers = []
      this.customAnswer = ''
      this.$emit('close')
    }
  },
  watch: {
    isOpen(newVal) {
      if (!newVal) {
        this.currentStep = 0
        this.answers = []
        this.customAnswer = ''
      }
    }
  }
}
</script>

<style scoped>
body {
  background: #3a393a;
}
</style>