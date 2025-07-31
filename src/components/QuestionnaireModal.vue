<template>
  <div class="flex flex-col items-center justify-center min-h-[80vh]">
    <!-- Logo at the top, always above the card -->
    <div class="z-20 flex justify-center" style="margin-bottom: -2.5rem;">
      <img src="/logo.png" alt="Logo" class="w-20 h-20 rounded-full border-2 border-white shadow-lg bg-white object-cover" style="object-position: center;" />
    </div>
    <!-- Card -->
    <div class="z-10 bg-white rounded-[2rem] shadow-xl w-[340px] max-w-full px-6 py-8 flex flex-col items-center">
      <!-- 1. Intro Card -->
      <div v-if="currentStep === 0" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">Sizni millionlar eshitadi va ko‘radi, hatto boshqa mamlakatlardan ham!</h3>
        <button @click="nextStep"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          Boshlash
        </button>
      </div>
      <!-- Dynamic Question Cards -->
      <div v-else-if="currentStep > 0 && currentStep <= questions.length" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">{{ questions[currentStep - 1].label }}</h3>
        <!-- Text Input -->
        <input v-if="questions[currentStep - 1].type === 'input'"
          v-model="answers[currentStep - 1]"
          :placeholder="questions[currentStep - 1].placeholder || ''"
          class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-4 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
        <!-- Radio/Checkbox Options -->
        <div v-if="['radio','checkbox'].includes(questions[currentStep - 1].type)" class="flex flex-col gap-3 w-full mb-4">
          <button v-for="option in questions[currentStep - 1].options" :key="option"
            @click="selectOption(option)"
            :class="optionButtonClass(option)"
          >
            {{ option }}
          </button>
          <!-- Boshqa (Other) logic -->
          <div v-if="questions[currentStep - 1].other && answers[currentStep - 1] === 'Boshqa'" class="w-full flex flex-col items-center">
            <input v-model="otherAnswers[currentStep - 1]" placeholder="Boshqa variantni kiriting"
              class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <button @click="nextStep" :disabled="!otherAnswers[currentStep - 1]"
              class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
              OK
            </button>
          </div>
        </div>
        <!-- Multi-select Options -->
        <div v-if="questions[currentStep - 1].type === 'multiselect'" class="flex flex-col gap-3 w-full mb-4">
          <button v-for="option in questions[currentStep - 1].options" :key="option"
            @click="selectOption(option)"
            :class="optionButtonClass(option)"
          >
            {{ option }}
          </button>
          <!-- Boshqa (Other) logic for multiselect -->
          <div v-if="questions[currentStep - 1].other && answers[currentStep - 1] && answers[currentStep - 1].includes('Boshqa')" class="w-full flex flex-col items-center">
            <input v-model="otherAnswers[currentStep - 1]" placeholder="Boshqa variantni kiriting"
              class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <button @click="nextStep" :disabled="!otherAnswers[currentStep - 1]"
              class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
              OK
            </button>
          </div>
        </div>
        <!-- Date Range -->
        <div v-if="questions[currentStep - 1].type === 'daterange'" class="w-full flex flex-col items-center mb-4">
          <div class="w-full mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Boshlanish sanasi va vaqti:</label>
            <input type="date" v-model="dateRange.fromDate" class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <input type="time" v-model="dateRange.fromTime" class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-4 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
          </div>
          <div class="w-full mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Tugash sanasi va vaqti:</label>
            <input type="date" v-model="dateRange.toDate" class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <input type="time" v-model="dateRange.toTime" class="w-full rounded-xl border border-gray-200 px-4 py-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
          </div>
        </div>
        <!-- File Upload -->
        <div v-if="questions[currentStep - 1].type === 'file'" class="w-full flex flex-col items-center mb-4">
          <input type="file" @change="onFileChange($event, currentStep - 1)" class="mb-2" />
          <button @click="nextStep" :disabled="!fileNames[currentStep - 1]"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            Davom etish
          </button>
        </div>
        <!-- Default Next Button (if not Boshqa) -->
        <button v-if="!isOtherActive(currentStep - 1) && questions[currentStep - 1].type !== 'file'"
          @click="nextStep" :disabled="!isStepValid(currentStep - 1)"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          Davom etish
        </button>
        <button v-if="currentStep > 1" @click="previousStep"
          class="mt-2 w-full max-w-xs bg-gray-200 text-gray-700 font-semibold py-3 rounded-full transition-colors duration-200">
          Orqaga
        </button>
      </div>
      <!-- Final Step: Summary/Confirmation -->
      <div v-else-if="currentStep === questions.length + 1" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">Barcha ma’lumotlarni tasdiqlang</h3>
        <ul class="w-full mb-4">
          <li v-for="(q, idx) in questions" :key="q.label" class="mb-2 text-sm text-gray-700">
            <span class="font-semibold">{{ q.label }}:</span>
            <span v-if="q.other && answers[idx] === 'Boshqa'"> {{ otherAnswers[idx] }}</span>
            <span v-else-if="q.type === 'file'"> {{ fileNames[idx] || 'Fayl tanlanmagan' }}</span>
            <span v-else-if="q.type === 'daterange'">
              {{ answers[idx]?.fromDate }} {{ answers[idx]?.fromTime }} - {{ answers[idx]?.toDate }} {{ answers[idx]?.toTime }}
            </span>
            <span v-else-if="q.type === 'multiselect'">
              {{ answers[idx]?.join(', ') }}
            </span>
            <span v-else> {{ answers[idx] }}</span>
          </li>
        </ul>
        <button @click="submitForm"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          YUBORISH
        </button>
        <button @click="closeModal"
          class="mt-2 w-full max-w-xs bg-gray-200 text-gray-700 font-semibold py-3 rounded-full transition-colors duration-200">
          INKOR
        </button>
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
  data() {
    return {
      currentStep: 0,
      questions: [
        { type: 'input', label: 'Ismingiz nima?', placeholder: 'Ismingizni kiriting' },
        { type: 'radio', label: 'Aksiya turi qanday?', options: [
          '1+1 aksiyasi', '2+1 aksiyasi', 'Diskont (narxlar pasayishi)', 'Rasprodaja (sotib tugatish)', '400 000 so‘mdan ko‘p xarid uchun sovg‘a', 'Har 10-mashinani yuvish bepul', 'Xarid uchun sovg‘a bor', 'Boshqa'
        ], other: true },
        { type: 'daterange', label: 'Aksiya amal qilish muddati va vaqti' },
        { type: 'radio', label: 'Qanchalik shoshilinch?', options: ['Shoshilinch emas', 'O‘rta shoshilinch', 'Juda shoshilinch'] },
        { type: 'radio', label: 'Aksiya mahsuloti turi', options: ['Xizmat ko‘rsatish / servis', 'Oziq-ovqat', 'Kiyim-kechak', 'Transport savdosi', 'Ko‘chmas mulk savdosi', 'Boshqa'], other: true },
        { type: 'checkbox', label: 'Aksiya amal qilish hududi', options: ['Yevropa', 'Osiyo', 'Afrika', 'Shimoliy Amerika', 'Janubiy Amerika', 'Avstraliya'] },
        { type: 'multiselect', label: 'Aksiya tili', options: ['O‘zbek', 'Ingliz', 'Turk', 'Fransuz', 'Nemis', 'Tojik', 'Hind', 'Ozarbayjon', 'Boshqa'], other: true },
        { type: 'radio', label: 'E\'londa asosiy urg‘uni nimaga qaratamiz?', options: ['Mahsulotga', 'Mahsulot soniga', 'Narxga', 'Sifatga', 'Aksiya shartiga', 'Brendga', 'Boshqa'], other: true },
        { type: 'radio', label: 'Radio orqali uzatiladigan habar qanday bo‘lsin?', options: ['Ha, audio rolik bo‘lsin', 'Yo‘q, boshlovchi og‘zaki e’lon qilsin', 'Farqi yo‘q'] },
        { type: 'radio', label: 'Siz ushbu mahsulot/brendning egasimisiz?', options: ['Ha, kompaniya rahbariman', 'Yo‘q, lekin vakolatim bor', 'Kampaniyaga aloqam yo‘q'] },
        { type: 'file', label: 'Yuridik hujjat (litsenziya, YATT, MCHJ) yuklang' },
        { type: 'radio', label: 'Telefon raqamini SMS orqali tasdiqlaysizmi?', options: ['Tasdiqlayman', 'Shart emas', 'Keyinroq'] },
        { type: 'radio', label: 'Ma’lumotlarni tasdiqlash', options: ['Ha', 'Yo‘q', 'Callback kutaman'] }
      ],
      answers: [],
      otherAnswers: [],
      fileNames: [],
      dateRange: {
        fromDate: '',
        fromTime: '',
        toDate: '',
        toTime: ''
      }
    }
  },
  methods: {
    nextStep() {
      // Save current step data if needed
      const currentIdx = this.currentStep - 1
      if (this.questions[currentIdx] && this.questions[currentIdx].type === 'daterange') {
        this.answers[currentIdx] = {
          fromDate: this.dateRange.fromDate,
          fromTime: this.dateRange.fromTime,
          toDate: this.dateRange.toDate,
          toTime: this.dateRange.toTime
        }
      }
      
      this.currentStep++
      console.log(this.answers)
    },
    previousStep() {
      if (this.currentStep > 0) this.currentStep--
    },
    selectOption(option) {
      const idx = this.currentStep - 1
      const q = this.questions[idx]

      if (q.type === 'checkbox') {
        if (!Array.isArray(this.answers[idx])) this.answers[idx] = []
        const i = this.answers[idx].indexOf(option)
        if (i > -1) this.answers[idx].splice(i, 1)
        else this.answers[idx].push(option)
      } else if (q.type === 'multiselect') {
        if (!Array.isArray(this.answers[idx])) this.answers[idx] = []
        const i = this.answers[idx].indexOf(option)
        if (i > -1) this.answers[idx].splice(i, 1)
        else this.answers[idx].push(option)
      } else {
        this.answers[idx] = option
        if (this.questions[idx].other && option !== 'Boshqa') {
          this.otherAnswers[idx] = ''
        }
      }
    },
    isOtherActive(idx) {
      return this.questions[idx].other && this.answers[idx] === 'Boshqa'
    },
    isStepValid(idx) {
      const q = this.questions[idx]
      if (q.type === 'input') return !!this.answers[idx]
      if (q.type === 'radio') return !!this.answers[idx] && (q.other ? (this.answers[idx] !== 'Boshqa' || !!this.otherAnswers[idx]) : true)
      if (q.type === 'checkbox') return Array.isArray(this.answers[idx]) && this.answers[idx].length > 0
      if (q.type === 'multiselect') return Array.isArray(this.answers[idx]) && this.answers[idx].length > 0
      if (q.type === 'daterange') {
        return !!this.dateRange.fromDate && !!this.dateRange.fromTime && !!this.dateRange.toDate && !!this.dateRange.toTime
      }
      if (q.type === 'file') return !!this.fileNames[idx]
      return true
    },
    optionButtonClass(option) {
      const idx = this.currentStep - 1
      const q = this.questions[idx]
      if (q.type === 'checkbox' || q.type === 'multiselect') {
        return [
          'bg-gray-100 text-gray-700 font-medium py-3 rounded-full w-full transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] border-2',
          (Array.isArray(this.answers[idx]) && this.answers[idx].includes(option)) ? 'border-[#FF2D6A] bg-[#FFF0F6]' : 'border-gray-100'
        ]
      } else {
        return [
          'bg-gray-100 text-gray-700 font-medium py-3 rounded-full w-full transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] border-2',
          (this.answers[idx] === option) ? 'border-[#FF2D6A] bg-[#FFF0F6]' : 'border-gray-100'
        ]
      }
    },
    onFileChange(e, idx) {
      const file = e.target.files[0]
      if (file) {
        this.fileNames[idx] = file.name
        this.answers[idx] = file
      }
    },
    submitForm() {
      this.currentStep++
    },
    closeModal() {
      this.currentStep = 0
      this.answers = []
      this.otherAnswers = []
      this.fileNames = []
      this.dateRange = {
        fromDate: '',
        fromTime: '',
        toDate: '',
        toTime: ''
      }
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
body {
  background: #3a393a;
}
</style>